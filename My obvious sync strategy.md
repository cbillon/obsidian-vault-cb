---
link: https://chadnauseam.com/coding/tips/my-obvious-syncing-strategy
---

# My "Obvious" Syncing Strategy 

I'm working on a flashcard web app to help teach myself French. Earlier versions of the app just saved all user data to localStorage. That is both annoying and scary. It's annoying because localStorage doesn't get shared with your other devices (I have two phones and two computers). And it's scary because you never know when localStorage will get evicted (I think iOS evicts it after a week of inactivity?).

So this weekend I decided to build a cloud sync layer for the flashcard app, and it went surprisingly well. Here were my requirements:

1. Local-first - I should be able to study french while on a plane or when taking the subway, and then have it sync up when I get online.
2. If I study french on my phone without internet, and then study french on my laptop without internet, it should not have a problem syncing this up when the devices go back online. (And it shouldn't ask me which device's data I'd like to throw away. I never want to throw away data!)
3. If I have two devices that are both online, whatever I do on one of them should be instantly reflected in the other. I don't want to have to actively press a "sync" button. I always want it to be synced!

I spent two days mulling the problem over in my brain, and then once I had a plan, I was able to implement it in a day. I had never done anything like this before, and the solution I came up with feels relatively "obvious" to me, so I'm sure it's not original. However, I thought I'd document it here in case anyone else is interested.

But first, here's the obligatory demo video. When I click the "remembered" or "forgot" button, that marks the current flashcard as reviewed. Watch how quickly it updates on the other browser!

Pretty good, right! I was proud of how well it worked. But how does it work?

## Device IDs 

We have multiple devices, so we need some way of referring to each of them. The easiest way is to store a UUID in local storage whenever you sign in on a device.

## The App Architecture 

The first (and possibly most difficult?) part of the process was to rewrite my app to be "event-based". By that, I mean that instead of user actions directly modifying the state, user actions cause an event to be appended an event vector. The app state is then recomputed based on the contents of that event vector. With only one device, that leads to the exact same outcome. But with multiple devices, it allows the app to perform the operation "insert this event into the middle of the event vector, and then recompute the state based on that". (This is pretty much the same idea as rollback netcode, if you're familiar with that.)

But how will we know _where_ in the event vector to insert foreign events? I decided to just give every event a timestamp, and then apply the events in the order implied by their timestamps. I'm sure there are more sophisticated options, but it seems to work fairly well in practice.

## The High-Level Plan 

Let's assume that our users (like all sane people) have 4 devices they might practice french on at any time.

![CleanShot 2025-05-27 at 23.00.52@2x.png](https://publish-01.obsidian.md/access/8a529a39ad1753175d73ccc6abc0547e/images/CleanShot%202025-05-27%20at%2023.00.52%402x.png)

We'll call them d1, d2, d3, and d4. The plan is for each device to produce "events" whenever the user takes an action, and then share them to the other devices via a central server.

Each device will maintain a list of events for each device, like this:

```
{
   d1: [event1, event2, event3, ...],
   d2: [eventA, eventB, eventC, ...],
   d3: [eventX, eventY, eventZ, ...],
   d4: [eventR, eventG, eventB, ...],
}
```

When a device produces a new event, that gets added to its own list. The goal is for each of the devices to agree on all the events that have been added.

Clearly, whenever a device appends to its list locally, it must let the central server know. But how does that event get to the other devices?

Well, each device can periodically ask the server "hey, I have 3 events from d1 and 4 events from d2. And I missing anything?" And then the server checks and realizes it has 10 events from d1 and 20 events from d2. It then responds with those events, and the sync is complete!

We can even implement this without a separate server outside the database. That server would be intuitively needed to handle adding new events and responding to requests for existing events. For adding events, Postgres has a feature called "row-level security" that allow devices to directly add data to the server without that being a massive security problem. For responding to requests for existing events, Postgres lets you add complex custom query logic that runs inside the database and it works great for this purpose.

But this is a pull-based mechanism. We can pull every 30 seconds, but that's not really fast enough for what I want (I want it to update instantly). To make it even faster, we can use Supabase's "realtime" feature to keep a Websocket connection open between each online device and the Supabase server. The devices tell Supabase they want to listen to new events, which makes Supabase notify them the instant any other device adds an event. The device can then immediately apply that event locally, instantly updating the state for the user. And that's it!

Well, that's it for syncing. For the app to be usable offline, we also have to write the data to disk. For this I used the "Origin Private File System", which is basically a special site-specific folder that your site can read from and write to. OPFS is great for this, because I can represent my sync state naturally. Each event is stored in `{user_id}/{device_id}/{event_index}.json`. Whenever we get a new event (either from a user action on the local device, or actions getting synced from another device) we just write it to a file in the appropriate folder. (The `user_id` folder allows you to log out of one account and log into another without events from one account getting intermingled with the other.)

![CleanShot 2025-06-06 at 14.49.58@2x.png](https://publish-01.obsidian.md/access/8a529a39ad1753175d73ccc6abc0547e/CleanShot%202025-06-06%20at%2014.49.58%402x.png)

But this is actually kind of annoying, because it means we have 3 representations to keep in sync: the in-memory state, the version on disk, and the version in the cloud. I'm sure that there are some annoying bugs hiding in here, particularly in the scenario where you have the app open in two different browser tabs while offline.

## The Details 

Hopefully you can get the high-level idea just from that. Now I'm going to go into the gory details of how I implemented this. Don't feel pressured to read this section. I'm honestly mostly writing it so I can refer back to it later.

### The Data Model 

As discussed previously, we have "events", which we use to update the app state:

```rust
// The type representing events
pub trait Event: Sized + PartialOrd + Ord + Clone + Eq + PartialEq {}

// The type representing state of your app. The app state 
// is derived by repeatedly applying the apply_event function.
pub(crate) trait AppState: Sized + Default {
    type Event: Event;

    // `Timestamped` will be explained later,
    // but you can think of it as an event annotated with a timestamp
    fn apply_event(self, event: &Timestamped<Self::Event>) -> Self;
}
```

Warning: most of the code examples will be Rust. Rust plays very nicely with React and I attribute it to why I'm able to write features so quickly for my app. (Including this complicated syncing code, which took me just one day). See my [rust-and-react](https://chadnauseam.com/coding/tips/rust-and-react) post for more details.

We have these events, the first thing we need to do is store them. I created a type called `EventStore` that is a map from `K` to `OrdSet<T>`, where `K` and `T` are any types you want. `OrdSet` is from the [im](https://docs.rs/im/latest/im/ordset/index.html) library, it's just an immutable ordered set.

```rust
#[derive(Clone, Debug, serde::Serialize, serde::Deserialize)]
pub struct EventStore<K: Eq + Hash, T: Ord + Clone> {
    pub events: HashMap<K, OrdSet<T>>,
}
```

An "ordered set" means that if you add `5, 1, 3, 2, 10` into your set and then iterate over it, you see them in the order `1, 2, 3, 5, 10`. But what's the hashmap for? Basically, I decided to maintain a separate set of events from each device. The HashMap keys are the device IDs, and the values are the sets of events produced by that device.

Let's move on to the type's methods. I added a convenience `add_event` function that takes a `(device_id, event)` and inserts the event into the appropriate HashMap entry, or creating a new entry if it doesn't already exist. The other function, `iter()` is more interesting. It implements ordered iteration over all the events in all the entries in the set – essentially joining them for you and hiding the fact that the events are split up by the originating device. Fun fact: converting n sorted arrays into one sorted array was the interview problem used at my dad's old place of work, and when I was first learning programming he let me give a whack at it to see how I'd do. So I was really getting back to my roots with this one.

```rust
impl<K: Eq + Hash, T: Ord + Clone> EventStore<K, T> {
    pub fn add_event(&mut self, key: K, event: T) {
        self.events.entry(key).or_default().insert(event);
    }

    pub fn iter(&self) -> impl Iterator<Item = &T> {
        // Collect all iterators from the OrdSets
        let mut iters: Vec<_> = self
            .events
            .values()
            .map(|set| set.iter().peekable())
            .collect();

        // Use a custom iterator that performs a k-way merge
        std::iter::from_fn(move || {
            // Find the iterator with the smallest current element
            let mut min_idx = None;
            let mut min_val = None;

            for (idx, iter) in iters.iter_mut().enumerate() {
                if let Some(val) = iter.peek() {
                    if min_val.is_none() || val < min_val.unwrap() {
                        min_idx = Some(idx);
                        min_val = Some(val);
                    }
                }
            }

            // Advance the iterator that had the minimum value
            if let Some(idx) = min_idx {
                iters[idx].next()
            } else {
                None
            }
        })
    }
```

The next thing I did was make a wrapper for events. My thinking was that I'll have some "domain-level" events like "the user reviewed a flashcard". In addition to that, I'll also have meta-events like "the user undid the previous event".

```rust
#[derive(Clone, Debug, Eq, PartialEq, Ord, PartialOrd, serde::Serialize, serde::Deserialize)]
pub enum EventType<E> {
    User(E), // domain-level event
    Meta(MetaEvent), // meta-level event
}

#[derive(Clone, Debug, Eq, PartialEq, Ord, PartialOrd, serde::Serialize, serde::Deserialize)]
pub enum MetaEvent {}
```

As you can see, I don't actually have any `MetaEvents` yet, but structuring it like this makes a difference because EventType is serialized with a `User` tag that will allow me to easily add `MetaEvent`s in the future without it being backwards incompatible.

What's next? Yep, another generic type! We'll use it to store the timestamp!

```rust
#[derive(Clone, Debug, serde::Serialize, serde::Deserialize, PartialEq, Eq, Ord, PartialOrd)]
pub struct Timestamped<E> {
    pub timestamp: chrono::DateTime<chrono::Utc>,
    pub within_device_events_index: usize,
    pub event: E,
}
```

You can see that I have the timestamp first. This is important, because the default `PartialOrd` and `Ord` implementations Rust will generate for you gives priority to each field according to its position in the struct declaration. So, having the timestamp first means that two instances of `Timestamped<E>` with different timestamps will be compared according those timestamps.

What comes after that is `within_device_events_index`. This isn't the best name ever, but basically, whenever a device creates an event it increments this number. This gives all events a stable ID they can be referred to by.

Now that we have that, let's wrap it up in a type that will also handle calculating the app state for us.

```rust
#[derive(Clone, Debug)]
pub(crate) struct Events<A: AppState> {
    events: EventStore<String, Timestamped<EventType<A::Event>>>,
    cached_state: A,
}
```

It does this in the most naïve possible way - whenever an event is added to the `events` field (which can only happen via an add_event method), we recalculate the cached_state from the beginning.

And tada! That's about it for the data model.

### The Server 

Now that we have all our events, we obviously need to store them on the server. Let's create a table in supabase:

```sql
-- create the `events` table
create table events (
  id bigserial primary key,
  user_id uuid references auth.users,
  device_id text not null,
  within_device_events_index integer not null
  event jsonb not null, -- the actual event payload
  created_at timestamptz
);

-- Create an index on events
create index idx_events_sync on events(device_id, within_device_events_index, user_id);

-- Allow users to add, remove, and modify events (although they should really only ever add them) 
create policy "Users can manage own events" on events
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);

-- make sure that there are never 2 events with the same (user_id, device_id, within_device_events_index). Our code should never produce this situation, so adding the constraints in the database serves as a sanity check
alter table events 
add constraint events_unique_device_index 
unique (user_id, device_id, within_device_events_index);

-- Allow "realtime" access to the database (a Supabase feature)
ALTER PUBLICATION supabase_realtime ADD TABLE events;
```

Hopefully that's all fairly self-explanatory. We basically just create a normal database table with a jsonb column to store our event pyaload.

The really fun part comes when we add a custom RPC function to Postgres. THAT is what allows us to sync!

```
create or replace function sync_events(last_synced_ids jsonb)
returns jsonb as $$
declare
  result jsonb = '{}'::jsonb;
  device record;
  known_devices text[];
begin
  -- Get array of known device IDs (will be NULL if empty)
  select array_agg(key) into known_devices
  from jsonb_each_text(last_synced_ids);
  
  -- First, get updates for known devices
  for device in select * from jsonb_each_text(last_synced_ids)
  loop
    result := result || jsonb_build_object(
      device.key,
      (
        select coalesce(array_to_json(array_agg(t.*)), '[]')::jsonb
        from (
          select * from events
          where device_id = device.key
          and within_device_events_index >= device.value::bigint
          and user_id = auth.uid()
          order by within_device_events_index
        ) t
      )
    );
  end loop;
  
  -- Then, get ALL events for unknown devices
  for device in 
    select distinct device_id 
    from events 
    where user_id = auth.uid()
    and (
      known_devices is null
      or device_id != all(known_devices)
    )
  loop
    result := result || jsonb_build_object(
      device.device_id,
      (
        select coalesce(array_to_json(array_agg(t.*)), '[]')::jsonb
        from (
          select * from events
          where device_id = device.device_id
          and user_id = auth.uid()
          order by id
        ) t
      )
    );
  end loop;
  
  return result;
end;
$$ language plpgsql security definer
set search_path = public, auth, extensions, pg_catalog;
```

What does this do? To explain that, think back to our EventStore data structure:

```rust
#[derive(Clone, Debug, serde::Serialize, serde::Deserialize)]
pub struct EventStore<K: Eq + Hash, T: Ord + Clone> {
    pub events: HashMap<K, OrdSet<T>>,
}
```

It's a map of device IDs to sets of events. So what if we take the length of each set? We'd get a map like `{d1: num_events_1, d2: num_events_2, ...}`. What this RPC function does is, we send that map to the server, and Supabase responds with all the events we're missing! Then we can add those to our local `EventStore`, and it's all finally done! (Well, we also have to query Supabase to send it events that it's missing, but that's fairly easily done.)

### Note on versioning 

I actually add a `V1` tag to all of the events I send to the server. That way if I ever decide to change the schema, I can bump the tag and write some migration code on the client.