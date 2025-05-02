---
link: https://fr.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094
byline: Sseleniumbootcamp
site: SlideShare
excerpt: Behat bdd training (php) course slides pdf - Download as a PDF or view
  online for free
twitter: https://twitter.com/@SlideShare
slurped: 2025-03-11T17:32
title: Behat bdd training (php) course slides pdf
---

- 1. [Behat and BDD](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#1) web training (PHP) 2 days

- 2. [Who is the](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#2) training for? • Ideal for testers wanting to gain BDD and automation testing experience • Some programming knowledge required but not essential • Testers involved with website testing • Testers wanting to gain technical skills • PHP programmers and Drupal programmers 2 www.time2test.co.uk

- 3. [High level • BDD overview • Step](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#3) Deﬁnitions • Behat History • Context Class • PHP Versions • Assertions • Installation • Command Line • Quick Usage Reference • Gherkin Language • Features • Mink Web Testing 3 www.time2test.co.uk

- 4. [What will you](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#4) learn? • Behaviour Driven Development Overview • Behat Overview and conﬁguration • Gherkin Language Explained • Syntax - Givens, Whens, Thens, And, But • Test Data • 4 Command Line usage • Understand Features, Scenarios, Step Deﬁnitions Context Class and Assertions • End to End Behat examples • • Mink for web testing www.time2test.co.uk

- 5. [schedule Day 1 Day 2 • php](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#5) primer • gherkin • behat background • case studies • mink api 5 www.time2test.co.uk

- 6. [PHP Primer](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#6)

- 7. [overview Lets have a](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#7) quick PHP programming recap 7 www.time2test.co.uk

- 8. [syntax PHP tags <?php …](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#8) ?> ! Comments with # 8 www.time2test.co.uk

- 9. [variables • All variables lead](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#9) with $ Integers $int_var = 12345; • assignments with = operator Doubles $doubles = 3.42 • • Boolean TRUE or FALSE constants variables don’t need to be declared Null $my_var = NULL; no need to specify the type e.g. String or int Strings $string = “This is some text”; 9 www.time2test.co.uk

- 10. [Constants • identiﬁer that can](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#10) not change • UPPERCASE • deﬁne(“NAME”, value); • echo constant(“NAME”); • Magic constants - ___LINE___, ___FILE___ e.t.c 10 www.time2test.co.uk

- 11. [Operators • Arithmetic Operators (](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#11) +, -, *, %, ++, —) • Comparison Operators ( ==, !=, >, <, >=, <=) • Logical operators ( and, or, &&, ||, !) • Assignment operators ( =, +=, -=, *=, /=, %= 11 www.time2test.co.uk

- 12. [Decisions • If… Else • elseIf • Switch 12 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#12)

- 13. [Loops for (initial; condition;](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#13) increment) { code to be executed; } while ( condition) { code to be executed; } do { code to be executed; } while ( condition); foreach (array as value) { code to be exe; } break continue 13 www.time2test.co.uk

- 14. [arrays • numeric arrays -](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#14) numeric index • associative array - strings used as index • multidimensional array - array of arrays 14 www.time2test.co.uk

- 15. [strings • single quotes versus](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#15) double quotes • string concatenation with a . • strlen($mystring) function • strpos($mystring, $searchstring) function 15 www.time2test.co.uk

- 16. [File Input/Output • fopen() -](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#16) open a ﬁle • ﬁlesize() - ﬁle length • fread() - read a ﬁle • fclose() - close a ﬁle 16 www.time2test.co.uk

- 17. [functions • functions • functions with parameters • functions](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#17) with return values • functions with default parameters 17 www.time2test.co.uk

- 18. [regular expressions • are a](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#18) sequence of pattern of characters • provide pattern matching • POSIX and PERL style matching 18 www.time2test.co.uk

- 19. [Exceptions Handling • try • throw • catch 19 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#19)

- 20. [Debugging • missing semicolons • not enough](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#20) equal signs • misspelled variable names • missing dollar signs • troubling quotes • missing parentheses and curly brackets • array index 20 www.time2test.co.uk

- 21. [date • time() - returns](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#21) seconds from 1 jan 1970 • getdate() - returns an associative array • date() - returns formatted string 21 www.time2test.co.uk

- 22. [Object Orientated Programming • class • object • member variables • member](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#22) functions • inheritance • constructors 22 www.time2test.co.uk

- 23. [class - $this • $this](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#23) - is a special variable and it refers to the same object i.e itself 23 www.time2test.co.uk

- 24. [objects • using new keyword • $travel](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#24) = new Books; • call member functions • $travel->setTitle(“travel to london”); • $travel->setPrice(100); 24 www.time2test.co.uk

- 25. [constructors • special functions which](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#25) are automatically called whenever an object is created/ initialised • __construct() to deﬁne a constructor with arguments 25 www.time2test.co.uk

- 26. [public, private and protected](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#26) members • public members are accessible insside, outside and in another the class to which is declared • private members are limited to the class its declared • protected members are limited to the class its declared and extended classes only 26 www.time2test.co.uk

- 27. [interfaces • common function names](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#27) to implementors who then develop code 27 www.time2test.co.uk

- 28. [constants • declared constants can](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#28) not change 28 www.time2test.co.uk

- 29. [abstract classes • abstract classes](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#29) can not be instantiated , only inherited 29 www.time2test.co.uk

- 30. [static • static members or](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#30) methods are accessible without needing an instantiation of the class 30 www.time2test.co.uk

- 31. [ﬁnal • ﬁnal methods can](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#31) not be overdided by child classes 31 www.time2test.co.uk

- 32. [parent constructors • sub classes](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#32) can extend the parent constructors. 32 www.time2test.co.uk

- 33. [Environment](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#33)

- 34. [overview • Mac • Windows • Unix • PHP 5 installed • PHP](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#34) IDE 34 www.time2test.co.uk

- 35. [PHP IDEs • Integrated Development](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#35) environments • many available -free and paid • Windows or Mac or Linux 35 www.time2test.co.uk

- 36. [Background](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#36)

- 37. [What is BDD? • human](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#37) readable stories • deﬁne business outcomes and drill into features • testing framework • Extends TDD - test driven development • Write a test that fails then watch it pass 37 www.time2test.co.uk

- 38. [Why BDD? • Deliver what](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#38) you client wants • Better communications and better collaboration • Extends the principles of TDD ( Test Data Driven) testing. 38 www.time2test.co.uk

- 39. [Behat History • Behat is](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#39) the PHP version of Cucumber • created by Konstantin Kudryashov (@everzet) 39 www.time2test.co.uk

- 40. [What does Behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#40) do? Scenario Step Given I have a ﬁle named “foo” regex Given /^I have a ﬁle named“([^”$/ Deﬁnition public function iHaveAFileNamed($ﬁle{ Do some work touch($ﬁle) Pass and Fal at each step unless an exception is thrown 40 www.time2test.co.uk

- 41. [Good BDD • practice • get into](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#41) the zone for creating features and scenarios • for web testing - understand the mink api 41 www.time2test.co.uk

- 42. [Quick Overview](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#42)

- 43. [overview • lets jump straight](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#43) into an end to end example using Behat and Mink 43 www.time2test.co.uk

- 44. [composer • dependency manager for](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#44) PHP external libraries 44 www.time2test.co.uk

- 45. [dependencies • download the dependencies](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#45) using a composer.json ﬁle • $php composer.phar install 45 www.time2test.co.uk

- 46. [behat —help • $>php bin/behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#46) —help 46 www.time2test.co.uk

- 47. [behat.yml • create this ﬁle](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#47) at root level • settings ﬁle • pronounce ( ya-mul ?) 47 www.time2test.co.uk

- 48. [initialise project • $>php bin/behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#48) —init • This will create some directories • +d features - place your *.feature ﬁles here • +d features/bootstrap - place bootstrap scripts and static ﬁles here • Also will create the ﬁle FeatureContext.php • +f features/bootstrap/FeatureContext.php - place your feature related code here 48 www.time2test.co.uk

- 49. [FeatureContext.php • Extend context from](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#49) MinkContext instead of the BehatContext 49 www.time2test.co.uk

- 50. [Feature Feature: Search In order](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#50) to ﬁnd an article As a website user I need to be able to search for am article 50 www.time2test.co.uk

- 51. [Execute Feature ! $bin/behat features/search.feature: • Magic](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#51) happens and test results shown • Good for headless browser without javascript 51 www.time2test.co.uk

- 52. [Selenium • Download selenium server](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#52) from seleniumhq • run selenium standalone server • java -jar selenium-server-standalone-2.38.0.jar • Tag your scenario with @javascript annotation 52 www.time2test.co.uk

- 53. [javascript test Run in](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#53) a browser that supports javascript @javascript 53 www.time2test.co.uk

- 54. [javascript execute ! • $bin/behat features/search.feature: • The](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#54) Firefox browser is invoked and the test execution is visible 54 www.time2test.co.uk

- 55. [implement a new](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#55) step execution will conveniently come back with a list of undeﬁned steps • /** * @Given /^I wait for (d+) seconds$/ */ public function iWaitForSeconds($arg1) { throw new PendingException(); } 55 www.time2test.co.uk

- 56. [new step deﬁnition](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#56) code // copy & paste at features/bootstrap/FeatureContext.php ! /** * @Given /^I wait for (d+) seconds$/ */ public function iWaitForSeconds($seconds) { $this->getSession()->wait($seconds*1000); } 56 www.time2test.co.uk

- 57. [Conﬁguration • behat.yml 57 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#57)

- 58. [Deﬁne a Feature • 4](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#58) line description • a line describes the feature • three following lines describe the beneﬁt, the role and feature itself 58 www.time2test.co.uk

- 59. [Deﬁne a Scenario • many](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#59) scenarios per feature • three line syntax • Given - describe the initial state • When - action the user takes • And Then - what the user sees after the action 59 www.time2test.co.uk

- 60. [Keywords • Given • And • When • Then 60 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#60)

- 61. [Execution • $> behat 61 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#61)

- 62. [Writing Step Deﬁnitions • behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#62) matches each statement of a scenario to a list of regular expression steps • Behat helps by creating an outline for undeﬁned steps 62 www.time2test.co.uk

- 63. [Regular expressions • Behat and](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#63) Mink rely on regex • here is a quick masterclass on regex 63 www.time2test.co.uk

- 64. [multi lines • triple quotation](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#64) syntax (“””) 64 www.time2test.co.uk

- 65. [Directory Structure • behat —init • features • features/bootstrap/](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#65) *.php ﬁles • features/bootstrap/featureContext.php 65 www.time2test.co.uk

- 66. [Further Details](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#66)

- 67. [Features Explained • features are](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#67) in a format called Gherkin • each feature ﬁle consists of a single feature • each feature will consist of a list of scenarios 67 www.time2test.co.uk

- 68. [Step Deﬁnitions • written in](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#68) php • consists of a keyword, regular expression and a callback • the regex is a method argument 68 www.time2test.co.uk

- 69. [Context Class • behat creates](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#69) a context object for each scenario 69 www.time2test.co.uk

- 70. [Assertions with PHPUnit // require_once](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#70) 'PHPUnit/Autoload.php'; require_once 'PHPUnit/Framework/Assert/ Functions.php'; // • assertEquals($numberOfRows, count($elements)); 70 www.time2test.co.uk

- 71. [Command Line Usage >>behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#71) —h - help options >>behat —v version >>behat —dl - view the step deﬁnitions >>behat —init - setup a features dir and featurecontext.php ﬁle 71 www.time2test.co.uk

- 72. [Gherkin Language](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#72)

- 73. [Overview • common language for](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#73) behaviour driven testing 73 www.time2test.co.uk

- 74. [Syntax • line orientated language • parser](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#74) divides input into features, scenarios and steps. 74 www.time2test.co.uk

- 75. [Features • each feature ﬁle](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#75) consist of a single feature • keyword feature: and three indented lines 75 www.time2test.co.uk

- 76. [Scenarios • each scenario begins](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#76) with keyword Scenario: followed by an optional scenario title 76 www.time2test.co.uk

- 77. [scenario outlines • template placeholders](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#77) for easy multi input 77 www.time2test.co.uk

- 78. [backgrounds • run before each](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#78) scenario • background: kewword 78 www.time2test.co.uk

- 79. [Steps • features consist of](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#79) steps: given, when, then 79 www.time2test.co.uk

- 80. [Given • known state before](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#80) the user or system interacts with the application under test 80 www.time2test.co.uk

- 81. [When • user action performed](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#81) on the system under test 81 www.time2test.co.uk

- 82. [Then • observe the outcome](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#82) and related to the feature description 82 www.time2test.co.uk

- 83. [And, But • Use and](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#83) and but for elegant and natural reading • not needed but good practice 83 www.time2test.co.uk

- 84. [Test Data • Tables used](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#84) for injecting data - different from scenario outlines • PyStrings - used for larger text input across many lines 84 www.time2test.co.uk

- 85. [tags • used to organise](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#85) features and scenarios 85 www.time2test.co.uk

- 86. [Mink - Web](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#86) Testing

- 87. [Overview • Mink is an](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#87) Extension to Behat • Think of a plugin 87 www.time2test.co.uk

- 88. [What is Mink? • Standalone](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#88) library to use PHP to command a browser • API to send commands to Selenium, Goutte, ZombieJS and more • The extension allows for BDD tests to be created without actually writing any PHP code 88 www.time2test.co.uk

- 89. [Installation • Update composer.json to](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#89) include • Mink • MinkExtension • Goutter adn Selenium2 drivers for Mink • Then update the vendor libraries ising • $>php composer.phar update 89 www.time2test.co.uk

- 90. [MinkExtension • behat.yml is the](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#90) behat conﬁg ﬁle • http://docs.behat.org/guides/7.conﬁg.html 90 www.time2test.co.uk

- 91. [MinkContext Extension • Gives us](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#91) access to the Mink Session to allow us to send commands to a browser • Inheritance of a pre-existing deﬁnitions • Before extending : we have 4 deﬁnitions • After extending : there are lots of deﬁnitions • Try it: $>php bin/behat -dl 91 www.time2test.co.uk

- 92. [wikipedia example • lets look](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#92) at a wikipedia example 92 www.time2test.co.uk

- 93. [First Example -](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#93) Mink headless lets look at a headless mink example 93 www.time2test.co.uk

- 94. [example - Mink](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#94) with Selenium • lets look at a javascript browser example 94 www.time2test.co.uk

- 95. [Mink api](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#95)

- 96. [Overview • http://mink.behat.org/api/index.html • Lets look at](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#96) some common Mink API 96 www.time2test.co.uk

- 97. [visit() • $this->visit('/'); 97 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#97)

- 98. [ﬁllField • ﬁllField('email', $email); 98 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#98)

- 99. [pressButton() • pressButton('Login'); 99 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#99)

- 100. [getSession() • $this->getSession(); 100 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#100)

- 101. [getPage() • getPage(); 101 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#101)

- 102. [ﬁndAll • css elements example • ﬁndAll(‘css’,’css_value_goes_here’); 102 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#102)

- 103. [ﬁndButton() • ﬁndButton(‘html_link_name’); 103 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#103)

- 104. [ﬁndButton()->click() • having found the](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#104) button element , you wish to click • ﬁndButton(‘html-link’)->click(); 104 www.time2test.co.uk

- 105. [locatePath() • based on current](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#105) session, now goto the deﬁned path • visit($this->locatePath(‘/user’)); 105 www.time2test.co.uk

- 106. [getStatusCode() • return the HTTP](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#106) status code from the current session • $session->getStatusCode(); 106 www.time2test.co.uk

- 107. [getCurrentUrl() • ->getCurrentUrl(); 107 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#107)

- 108. [Next Steps](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#108)

- 109. [Practice • Try out the](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#109) different mink web emulators • Sublime extension • phantom.js • Symfony • Drupal extension 109 www.time2test.co.uk

- 110. [case studies](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#110)

- 111. [Wordpress casestudy • Wordpress Behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#111) Mink Context Example 111 www.time2test.co.uk

- 112. [case study 2 • Lets](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#112) look at an example - behat with mink 112 www.time2test.co.uk

- 113. [case study 3 • Lets](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#113) look at an example - behat ,mink , website 113 www.time2test.co.uk

- 114. [case study 4 • google](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#114) search 114 www.time2test.co.uk

- 115. [case study 5 115 www.time2test.co.uk](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#115)

- 116. [case study 6 • features](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#116) master class 116 www.time2test.co.uk

- 117. [case study 7 • ls](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#117) example on unix 117 www.time2test.co.uk

- 118. [case study 8 • open](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#118) scholar project • good established behat mink framework 118 www.time2test.co.uk

- 119. [case study 9 • Behat](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#119) + Mink demo github 119 www.time2test.co.uk

- 120. [Conclusions](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#120)

- 121. [goals and objectives • Review](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#121) your goals. • Have we met your expectations? • Email us with your feedback 121 www.time2test.co.uk

- 122. [Thank you • From the](https://www.slideshare.net/slideshow/behat-bdd-training-php-course-slides-pdf/29036094#122) Time2test Team 122 www.time2test.co.uk