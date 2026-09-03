---
tags:
  - moodle
  - release
---
Yeah, in an "update/upgrade" discussion one has to use certain terms consistently. Wisecat uses the same terms as in [https://moodledev.io/general/releases](https://moodledev.io/general/releases), which makes sense.  
  
For example, limiting ourselves to existing releases (not future releases),  
- 4.5, 5.0, 5.1,.. are major releases, what the community casually call Moodle versions  
- 4.5.0, 4.5.1,.. 4.5.13, 5.0.0, 5.0.1,.. 5.0.9 are minor or point releases or just releases.  
- Any transition, say from 4.5.x to 4.5.y are updates  
- Every transition, say 4.5.x to 5.0.y, or from 4.5.x to 5.1.z are upgrades

From Moodle 
These are the target dates for releases. These dates may vary slightly due to unforeseen circumstances.

| Release type                                                                                            | Frequency | Months                                           |
| ------------------------------------------------------------------------------------------------------- | --------- | ------------------------------------------------ |
| [Major](https://moodledev.io/general/development/process#major-release-cycles) (eg. 3.x)                | 6 monthly | April, and October                               |
| [Minor](https://moodledev.io/general/development/process#stable-maintenance-cycles) (Point) (eg. 3.x.y) | 2 monthly | February, April, June, August, October, December |


# Code Base as chain 

### Develop the code using Git
**The codebase is a chain, of which each plugin is a link.**

Develop your code on an open Git repository, like github.com. That enables people to see your code and to help you as it develops. Testers and early adopters also have the opportunity to try it early in the process and give you more valuable feedback.

Coverage with automated tests ([PHPUnit](https://docs.moodle.org/dev/PHPUnit) or [Behat integration](https://docs.moodle.org/dev/Behat)) is mandatory for new features.

It is essential that your code follows the [Moodle Coding Guide](https://moodledev.io/general/development/policies).