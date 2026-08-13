## Moodle :
 
 - application LAMP source PHP web server base de données  - source PHP - vcs git version X.Y.Z
   Moodle suit  a peu pres le modele semver
- X serie les depreciations deviennent effective lors du chagement de série
	- un plugin fonctionnant pour une version Moodle, foctionnera pour les versions suivantes de la série
	- X.Y version majeure introduction new features, corrections
	- .X.Y.Z corections de la version X.Y
	- one branch for each major version
	- each release coresponds  a tag
## Plugins
- apporte des fonctionnalites supplémentaires
- nom normalisé  <<type_name>
-  version.php : metadata du plugin

### role du catalogue

-  administation du plugin au cours de sa vie
	-  critéres d'acceptation
	-  unicité des noms 
	- changement de mainteneur,  
	

## CodeBase
A codebase is ==the complete collection of source code, configuration files, and assets maintained together to build a software system==. It goes beyond raw source code by including documentation, build scripts, and dependencies.
[1](https://www.sonarsource.com/resources/library/code-base-in-software-development/)
[2](https://en.wikipedia.org/wiki/Codebase)
[3](https://www.quora.com/What-is-the-difference-between-codebase-and-source-code)

Core Components

- **Source code**: human-readable instructions written in programming languages.

- **Configuration**: settings required to run or compile the program.

- **Tooling and tests**: scripts used to automate deployment and verify functionality.
	[1](https://en.wikipedia.org/wiki/Codebase)
	[2]](https://www.sonarsource.com/resources/library/code-base-in-software-development/)

### installation d'un plugin

- simplemnt recopie du sour du plugin dans un répertoire qui dépend du type de plugin


from ![Avatar James Steerpike|32](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

### Composer install of dependencies & superuser account

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5)
Installing composer as root could allow plugins to do naughty stuff. A solution is to create a new user specifically for composer and add the web server to the group.

AI suggested something like this (The web server may be apache and not www-data if you are on Redhat based )  You could also set your regular no sudo account as the one installing composer.

# Create a deploy user if you don't have one  
sudo useradd -m -s /bin/bash deploy

# Run composer as the deploy user  
sudo -u deploy composer install --no-dev  --classmap-authoritative

sudo chown -R deploy:www-data vendor
sudo chmod -R 644 vendor

see also https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md

Please see:

[https://docs.moodle.org/502/en/Installing_plugins](https://docs.moodle.org/502/en/Installing_plugins)

[https://moodledev.io/general/development/tools/composer](https://moodledev.io/general/development/tools/composer)

As well as:

[https://docs.moodle.org/502/en/Composer_vendor_directory_not_found](https://docs.moodle.org/502/en/Composer_vendor_directory_not_found)

Moodle's routing system uses a **front controller** pattern (`r.php`) rather than being "front loading". Introduced in Moodle 5.1 and enhanced with strict configuration checks in Moodle 5.2, the router directs web traffic through a central script when clean URLs or specific components are called. [[1](https://moodle.org/mod/forum/discuss.php?d=473950), [2](https://docs.moodle.org/en/Configuring_the_Router), [3](https://pimenko.com/en/moodle-5-2-new-features-2026/), [4](https://www.cmsgalaxy.com/blog/complete-tutorial-upgrade-moodle-to-5-2-on-cpanel-shared-hosting-step-by-step/), [5](https://moodle.org/mod/forum/discuss.php?d=474506)]

Router and Front Controller Setup

- **Front Controller:** Moodle uses `r.php` in the main directory as the router target handler.

- **Web Root:** The server document root must point to the `/public` directory.

- **Server Rules:** Handlers like `FallbackResource /r.php` or URL rewrite rules map requests that do not match physical files directly into the routing system.

- **Environment Check:** Moodle 5.2 includes a specific system check to verify that your web server and router are set up correctly. [[1](https://www.cmsgalaxy.com/blog/complete-tutorial-upgrade-moodle-to-5-2-on-cpanel-shared-hosting-step-by-step/), [2](https://moodle.org/mod/forum/discuss.php?d=473950), [3](https://docs.moodle.org/en/Configuring_the_Router), [4](https://pimenko.com/en/moodle-5-2-new-features-2026/)]


    
  
    
    ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFEAAABRCAYAAACqj0o2AAAQAElEQVR4AeybB2BVRbrHf+ckubnpvRfSOyFAkI6AgOgqu6y6rrq6uyp29731ucVnF8sTFV0bi66Luq6yVkAFKSK9SgIYSCippN3Um5ub28ubc0JCzMN9uyYW8A73mzPzzTffzPc/38w58x0iuz1pyAjIfEfJ7QaFvtHh3U6cLjHQMAxic7i+Uou845OPuO66Bdx66y3cveivOIdnTHXAw7s2cMN11/Ob22/j4WeWcmD3agonzeBg2WF+Mmcam4/UqnJDyexWMwcPlGC22PrVuJwOVi57hpH5ubyyams//+sWjJ31JMQks2pLyWlVyIfK9tNk1nL1tTfzs9nj0Le30NjcQktzM2ajidraWlFuxGI2YbfZqKquoUnXqipzWSycaNSp7e16A40N9RjNp4wxdXWwa+chbr9hAfPmzCA1ezwP/uEOokL8sJh6VB24XeiaGtW+vYzevG+sjo5OqmrrsDtdaoOpq4XjVdXoO9vV+u71KzlvzkWUVxyhz1v09ZW8vnwl9yx8gstnFeOwmamuqVHtUjrpO9qoa2hWirS0ttDQ2ISlu52G5jYxlwZ0Hd1qm5I1NpygsUmRPcVT+ANJXc49JgP1NZV0SUFs/+BVzr/4pzz62EJ2bl7JpFkXcvfvfs/2XXu48aqf87t7H+bKa37NslXbOXroc348axb33i9k95bw3EN38XFp2UD92CUXje3tdBiMnDhWxl2PLUbfbemX+ftflvDHBx/nrj/+kSVvrOrnN1QdYsac+fz+vof42U9+wrtrd1NVtpf5F1/NwoUL+dVNd3C0upXdez/H5dbwyvIVdBqt6h6xf+82DtW28fLSJRw6XMFNv7iKux9exNW/vJa/fLCFI/u2cemVv0JZ6vf91w0sWb6BNa89xYWXXc1zTz7KzFmz2VPeILBYxnXX38JDTy6lxxHcP7fBBRVEXVMda1a9R31d7/IKD4/i0UXPkhjhQ6BfAAuffQmaD7Kt3MyrS5/jN1fP4Z5HnqDHYsct+fPIE89w0ezpPPjUn7ls4mgGJpOtm9eXLWP7/kpcLie1bXrsJ/cpLTIvPvMIzTodDouef/z9VSHTu584XS70ki833vpbrpg9ht0lB1m3YjmhY+ay9KWXyQ4y8/7atUyfPh1vbxf33HEbMaF+IElMmjufManx3P34Eux1e1m/v4fnFz/Fbb+Yw3PPv4hVrBaTrXecLmGDw4maQiMzuOeBh4g1t7Ny50GeeOAOpl5+My8+fjf+Xl2qzOkyFcTiCTN56NHHmTs5XzUiPDQIf18vVV7r6yvK3gSGhGB1tdLUaeb40eOkpmXj7SUjCYoM9hUe4OTAsUrausxqv74sKiiOV177G3ffdFkfq/9qw0VYYASTZvyIhx99hhcWP4YsS/3tSiHAR1YuKkWER1BdfQR9Wwu6Lgf+2hCUh5PdYafdYOpf8qrwySwgOAi7o0mshB6qyquJjokkLDwAL0s37S06jB1tJyVBHVr2QatwHODjH0FnXQ2tjQ1iKxM2KvzTkBwdH0/rkZ1cceWV3HDrQnwCw8hIS1FFff1Cyc/tBav43Eu4Yf5Ufnf79eyp7OKF+28hICiYpPwClKRs8CuWPcu+yhqlqlJgUAg5eVlqWcl8hVcXi7qfVkt6RiZ+Wg133Hcvu9e/w6LH/4dd5U2KmEoaXz/G5qWj0WiIio0nIS6K6fN/wdhoB1defQ2+SXlcPm8mabmFXHLuaB5a/DJtBova18vbh3QxTkiAL6OnzOPWK2Zzx20L2FnZzKJ77iQtp4hZxalce+d9BESmEhsdTkR0IlkZcUiyF5mFIxkRF8l9jz3NwU3v8+jSdykaM1asShVedYyBmTzvil+zZvVHbPpsI2+/+TQXXrmARY8+qMqMKDyPN159SfVEr4BA/vvhxXzw3ru8/carjMoZISZaxAd/X6rK+mgDefhPLzN3TJ5aV7KciXN4/fUXlKJKmSPPYfOKt8jMSOP5ZW9SnBLHzB/9jFUrV/DCSy9x7U+nq3JKlpSey4b3/0Z6cjRX//YB7rx2PlFxybwolvKGdWv581MLiREeFS54L739Ecufv5+4MH+lKz7aILGP/YkxWfF4+SvzfpIV77/HO8vfZHReigAugSdfeoNVry/lrbfe5D+uuZCpl93MksfvQrnRT7/1LgsuKmbk9ItZs2Edf3lhMZ9tWs2M4lxV/+BMliRJLCG5nyRJEtuKhJpEWZZltahkkiSpcpLU2y5JvXVOJkmSRN+TFeUi6qfrrzQpfEkpCFLKCvXVBUv8pN6xQOiUBfW2SpLUyxdXTiZJUubf236ShSR4p8oSqv4v9VF4EpLUq1uSRF0QIknCZkmSRAn6+vVdVeagTB5U91S/BgIeEL8GaIO7nA0gDrbpW6/L5toqPDQ0DGSXsRsPDQ0D5bGmPII8JJ7I4lH8tXDw7InDsIN6QPSAOAwIDIMKjycOB4iSJP3LalxOp4hm2P5feUk6vU5J6uVLUu/1nymSpF4ZSTp1tdvtp7oo4RtRk6RT7Y6B7aLt//xEH0k6JT+43WwyqSxJ6pVRK/9CJnd16rFZrSo4TptFhJbcdHd14XLYcDocos0mwmMueoxGutt0VByrEw8xWW3v6uwUUWMbJtFm6e4WcTozNhEBN3QZcArAewwGLObe0JjFZMYg5JU5dXV0YLNbsYhJG7tFCEtEzN1ivB6zE6Poa7VaaG9pxdRtxKDXI4l/ehHh3rVjk4hFeuMwNLNu1QbxScBER2srDhH1tgkdR49UqfPqbO/otUfMX5m38ulAkiT2bd2CW+OLuceEYrcyF/GVCoeQc1pN1FbVIflqVfuNPTZhpxcWs0W1RZFzixinYrNLjCdJEm4x5y4xL7m16QR7Nm+jobaO6tKd7NtdypFDx7C0ldFU38zB7VtR7tDBnVs4UtWmjIskSWx+5zW2bVjDrk/X8XlpNfu2befo0Sr2bNqCTkTJS7btYN++IygAOMSngKNHjnGiooz65gpqK+toaj1BxZFKzIZOSnfupaV2L3t3fcFHG7azb882mkTI/tMP3mf/zl0c2L6Jtg49co9VHd/gsKiG7Vy3grbmZjav3Ur1gc85cbScprpN1De0sm/XTnZv3io+CdSx4dNSAYiMsceOrauTY0craa4sw+6UcVu7OFyyn40rVtIibu7+bds4tGMLn7z5hui/jXWrN1NfdRS73YkkSTRWHeeTdSWqvpLSNTS2tCFHRkbRIxQZhfcZus3EJyfi6OmgosEq7rSVLnHXNq3eSFBYAPXidKNY4RZe1tHUyshzppOUHENETAh+fm7iYyMxa8KJFbE/5ZuIX3AwfgH+wrtdhIQE4y/ij8aWEwSIAK+/25fA4DCCgvyEx7UJL28mUGMnJjKJbncQaRkj6HZoGDt1CpK1noDgCHwDe+N54X4afLxAo40lfES6ALeedkcAwRFxOJxWQiNikEzt4nOBTHRgAJLdJObgRuMl0yN7ixilL36hYbQKL/bShuLsqMOkjcPXbUfSuIhOz2PWj+dSvu1TEiJ8Obh1h+ijob7yGO1GL7Fa9AoMWJ0SIUGhyMGR0WScMxZfbzdWr2AsRgPewVEUF0/EotcRm5DE+XNG0WaVuGDeHDJS48UnAYkfXXcz9UcOESuMyBLxwcSsIkJjEiiKsVFS1kjxlIlkp8eoW4FWBGdd3W2YbJA/ei46XY3wAjA011NWXsv48SNFxDxEjTiPF8HS4pxU/PyDKJycS

[Configuring the router](https://docs.moodle.org/502/en/Configuring_the_Router)

ftom [catalyst](https://www.catalyst-eu.net/blog/2024/04/02/the-xz-backdoor-and-moodle)
One area that should create pause for thought is third-party plugins. The [Moodle Plugins database](https://moodle.org/plugins) contains over 2000 plugins, including many that haven’t been updated in several years. It’s conceivable that a malicious party could pressure a maintainer of a popular plugin who is no longer able to work on it to hand maintainership over to them. They could then follow a similar path to the XZ exploit, releasing a compromised version and encouraging users to update.

When adding plugins to your Moodle site, it is important to perform due diligence and ensure that the plugin is being well-maintained. A well-maintained plugin will have regular updates from a variety of developers, meaning the pressure isn’t piled on one individual. Bug reports will be responded to, and Pull Requests will be merged or rejected. If a project has financial sponsorship or commercial backing, the developers can dedicate more of their time to it. In the Moodle community, you can often meet developers at in-person MoodleMoots or online events, so you may be able to associate on-screen names with real individuals.

With a site that uses plugins, you should also perform regular reviews to ensure the due diligence you performed at the start still holds up. Does the plugin have a release for the current Moodle version? Is it still actively maintained? Has it changed hands since you first installed it?

Catalyst can help with all of this. We offer a comprehensive plugin review service which will assess the quality and security of the code itself, but also the health and sustainability of the project that produces it.

During upgrades, we can help review your plugins and remove any that have become obsolete or aren’t well maintained. If an unmaintained plugin is crucial to your organisation, the open source nature of Moodle’s ecosystem allows Catalyst to take over maintenance on your behalf, ensuring you stay secure into the future.