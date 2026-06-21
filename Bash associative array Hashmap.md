---
tags:
  - bash
  - array
---
Hashmaps (associative arrays) are a great way to store a bag of key-value data. At work I was writing something that needed me to spawn a bunch of GPU instances, GPU availability is spread out by region and GPU type. I wanted to store a mapping of GPU kind to region name and for some reason I thought it would be a good idea to do it in bash. I was horrified to find out that bash has hashmaps, and decided to write this note to tell you how to use them.

Here's how you do it:

First, declare the hashmap with the declare builtin:

# -A means associative array (hashmap)
declare -A FLY_REGIONS
Then add some values into it from the fly.io documentation:

FLY_REGIONS["a100-40gb"]="ord"
FLY_REGIONS["a100-80gb"]="mia"
FLY_REGIONS["l40s"]="ord"
You can also predeclare the array with values in a read-only state:

# -A means associative array (hashmap)
# -r means read-only
declare -A -r FLY_REGIONS=(
  ["a100-40gb"]="ord"
  ["a100-80gb"]="mia"
  ["l40s"]="ord"
)
You can look up values with the same syntax you used to set them:

echo "${FLY_REGIONS["a100-40gb"]}" # ord
You can iterate over the keys with a for loop on the @ value:

for size in "${!FLY_REGIONS[@]}"; do
  echo "size: $size, region: ${FLY_REGIONS[$size]}"
done
You can iterate over the values with a for loop on the * value:

for region in "${FLY_REGIONS[*]}"; do
  echo "region: $region"
done
You can delete individual keys with unset:

unset FLY_REGIONS["a100-40gb"]
If you need to delete the entire hashmap, you can also do it with unset:

unset FLY_REGIONS
Please don't use this for evil.