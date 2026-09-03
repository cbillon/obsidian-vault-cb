---
link: https://moodle.atlassian.net/browse/MDL-85109
---
There are some unit tests that appear to be designed to verify that plugins implement some APIs correctly, for example the Privacy API, and SVG icons.

It would be really useful if this sort of test had a tag so when we are writing or upgrading a plugin we can easily run all of these unit tests, when we run the plugins own tests, to help verify that everything is working correctly.

Some of the tests that I have seen that do this sort of thing are:

- [core_privacy\privacy\provider_test::test_null_provider](https://github.com/moodle/moodle/blob/MOODLE_405_STABLE/privacy/tests/privacy/provider_test.php#L87 "https://github.com/moodle/moodle/blob/MOODLE_405_STABLE/privacy/tests/privacy/provider_test.php#L87")
    
- [core_privacy\privacy\provider_test::test_all_providers_compliant](https://github.com/moodle/moodle/blob/MOODLE_405_STABLE/privacy/tests/privacy/provider_test.php#L177 "https://github.com/moodle/moodle/blob/MOODLE_405_STABLE/privacy/tests/privacy/provider_test.php#L177")
    
- [core\output\icon_system_fontawesome_test::test_svg_fallback](https://github.com/moodle/moodle/blob/main/lib/tests/output/icon_system_fontawesome_test.php#L32 "https://github.com/moodle/moodle/blob/main/lib/tests/output/icon_system_fontawesome_test.php#L32")
    

So for example being able to run a unit test commend like: v`endor/bin/phpunit --group api-verification` would make it easy to use them all
 
Petr Skoda

10 juin 2025 à 18:06

hi @Neill Magill, there is a patch relevant to this issue in my last comment in [MDL-85666: PHPUnit install should detect missing strings or bad capability namesClosed](https://moodle.atlassian.net/browse/MDL-85666#icft=MDL-85666).

There is one unsolved problem related to performance - if we run tests with data provider listing all components then it takes at lease 1 minute to the tests repeatedly for all plugins, but if we do the same thing as test_svg_fallback(), then you cannot use --filter=mod_myplugin to limit execution to one plugin only.