---
tags:
  - dockerfile
---
The entrypoint script does run after the volume mount. I encountered a similar issue but it was actually due to using single quotes around the entrypoint instead of double quotes. Because of this, the container was falling back to using the default entrypoint of `/bin/sh`. As your question is already answered, I'm leaving this for others who end up here via Google.

This:

`ENTRYPOINT ["entrypoint.sh"]`

Not this:

`ENTRYPOINT ['entrypoint.sh']`