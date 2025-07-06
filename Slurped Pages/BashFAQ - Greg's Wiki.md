---
link: https://mywiki.wooledge.org/BashFAQ
excerpt: "Note: The FAQ was split into individual pages for easier editing.  Also, for faster loading of this page, the answers are no longer presented here in their entirety."
slurped: 2025-07-01T18:43
title: BashFAQ - Greg's Wiki
tags:
  - bash
  - tips
---

## BASH Frequently Asked Questions

**Note**: The FAQ was split into individual pages for easier editing. Also, for faster loading of this page, the answers are no longer presented here in their entirety.

Readers, click the

[BashFAQ/nnn]

link at the bottom of each answer to read the rest of the answer.

Editors, click the '[edit]' link at the bottom of each entry. Don't add new ones to this page; create a new subpage with the next available question number instead.

Thank you.

These are answers to frequently asked questions on channel #bash on the [irc.libera.chat](https://libera.chat/) IRC network. These answers are contributed by the regular members of the channel (originally heiner, and then others including greycat and r00t), and by users like you. If you find something inaccurate or simply misspelled, please feel free to correct it!

All the information here is presented without any warranty or guarantee of accuracy. Use it at your own risk. When in doubt, please consult the man pages or the GNU info pages as the authoritative references.

[BASH](https://mywiki.wooledge.org/BASH) is a [BourneShell](https://mywiki.wooledge.org/BourneShell) compatible shell, which adds many new features to its ancestor. Most of them are available in the [KornShell](https://mywiki.wooledge.org/KornShell), too. The answers given in this FAQ may be slanted toward Bash, or they may be slanted toward the lowest common denominator Bourne shell, depending on who wrote the answer. In most cases, an effort is made to provide both a portable (Bourne) and an efficient (Bash, where appropriate) answer. If a question is not strictly shell specific, but rather related to Unix, it may be in the [UnixFaq](https://mywiki.wooledge.org/UnixFaq).

This FAQ assumes a certain level of familiarity with basic shell script syntax. If you're completely new to Bash or to the Bourne family of shells, you may wish to start with the [BashGuide](https://mywiki.wooledge.org/BashGuide).

More advanced users may wish to read [BashPitfalls](https://mywiki.wooledge.org/BashPitfalls) and [BashProgramming](https://mywiki.wooledge.org/BashProgramming).

If you want to help, you can add new questions with answers here, or try to answer one of the [BashOpenQuestions](https://mywiki.wooledge.org/BashOpenQuestions).

Chet Ramey's official [Bash FAQ](http://tiswww.case.edu/php/chet/bash/FAQ) contains many technical questions not covered here.

Contents

1. [How can I read a file (data stream, variable) line-by-line (and/or field-by-field)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F001.How_can_I_read_a_file_.28data_stream.2C_variable.29_line-by-line_.28and.2For_field-by-field.29.3F)
2. [How can I store the return value and/or output of a command in a variable?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F002.How_can_I_store_the_return_value_and.2For_output_of_a_command_in_a_variable.3F)
3. [How can I sort or compare files based on some metadata attribute (most recently modified, size, etc)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F003.How_can_I_sort_or_compare_files_based_on_some_metadata_attribute_.28most_recently_modified.2C_size.2C_etc.29.3F)
4. [How can I check whether a directory is empty or not? How do I check for any *.mpg files, or count how many there are?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F004.How_can_I_check_whether_a_directory_is_empty_or_not.3F__How_do_I_check_for_any_.2A.mpg_files.2C_or_count_how_many_there_are.3F)
5. [How can I use array variables?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F005.How_can_I_use_array_variables.3F)
6. [How can I use variable variables (indirect variables, pointers, references) or associative arrays?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F006.How_can_I_use_variable_variables_.28indirect_variables.2C_pointers.2C_references.29_or_associative_arrays.3F)
7. [Is there a function to return the length of a string?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F007.Is_there_a_function_to_return_the_length_of_a_string.3F)
8. [How can I recursively search all files for a string?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F008.How_can_I_recursively_search_all_files_for_a_string.3F)
9. [What is buffering? Or, why does my command line produce no output: tail -f logfile | grep 'foo bar' | awk ...](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F009.What_is_buffering.3F__Or.2C_why_does_my_command_line_produce_no_output:_tail_-f_logfile_.7C_grep_.27foo_bar.27_.7C_awk_...)
10. [How can I recreate a directory hierarchy structure, without the files?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F010.How_can_I_recreate_a_directory_hierarchy_structure.2C_without_the_files.3F)
11. [How can I print the n'th line of a file?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F011.How_can_I_print_the_n.27th_line_of_a_file.3F)
12. [How do I invoke a shell command from a non-shell application?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F012.How_do_I_invoke_a_shell_command_from_a_non-shell_application.3F)
13. [How can I concatenate two variables? How do I append a string to a variable?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F013.How_can_I_concatenate_two_variables.3F__How_do_I_append_a_string_to_a_variable.3F)
14. [How can I redirect the output of multiple commands at once?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F014.How_can_I_redirect_the_output_of_multiple_commands_at_once.3F)
15. [How can I run a command on all files with the extension .gz?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F015.How_can_I_run_a_command_on_all_files_with_the_extension_.gz.3F)
16. [How can I use a logical AND/OR/NOT in a shell pattern (glob)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F016.How_can_I_use_a_logical_AND.2FOR.2FNOT_in_a_shell_pattern_.28glob.29.3F)
17. [How can I group expressions in an if statement, e.g. if (A AND B) OR C?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F017.How_can_I_group_expressions_in_an_if_statement.2C_e.g._if_.28A_AND_B.29_OR_C.3F)
18. [How can I use numbers with leading zeros in a loop, e.g. 01, 02?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F018.How_can_I_use_numbers_with_leading_zeros_in_a_loop.2C_e.g._01.2C_02.3F)
19. [How can I split a file into line ranges, e.g. lines 1-10, 11-20, 21-30?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F019.How_can_I_split_a_file_into_line_ranges.2C_e.g._lines_1-10.2C_11-20.2C_21-30.3F)
20. [How can I find and safely handle file names containing newlines, spaces or both?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F020.How_can_I_find_and_safely_handle_file_names_containing_newlines.2C_spaces_or_both.3F)
21. [How can I replace a string with another string in a variable, a stream, a file, or in all the files in a directory?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F021.How_can_I_replace_a_string_with_another_string_in_a_variable.2C_a_stream.2C_a_file.2C_or_in_all_the_files_in_a_directory.3F)
22. [How can I calculate with floating point numbers instead of just integers?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F022.How_can_I_calculate_with_floating_point_numbers_instead_of_just_integers.3F)
23. [I want to launch an interactive shell that has special aliases and functions, not the ones in the user's ~/.bashrc.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F023.I_want_to_launch_an_interactive_shell_that_has_special_aliases_and_functions.2C_not_the_ones_in_the_user.27s_.2BAH4-.2F.bashrc.)
24. [I set variables in a loop that's in a pipeline. Why do they disappear after the loop terminates? Or, why can't I pipe data to read?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F024.I_set_variables_in_a_loop_that.27s_in_a_pipeline._Why_do_they_disappear_after_the_loop_terminates.3F_Or.2C_why_can.27t_I_pipe_data_to_read.3F)
25. [How can I access positional parameters after $9?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F025.How_can_I_access_positional_parameters_after_.249.3F)
26. [How can I randomize (shuffle) the order of lines in a file? Or select a random line from a file, or select a random file from a directory?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F026.How_can_I_randomize_.28shuffle.29_the_order_of_lines_in_a_file.3F__Or_select_a_random_line_from_a_file.2C_or_select_a_random_file_from_a_directory.3F)
27. [How can two unrelated processes communicate?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F027.How_can_two_unrelated_processes_communicate.3F)
28. [How do I determine the location of my script? I want to read some config files from the same place.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F028.How_do_I_determine_the_location_of_my_script.3F__I_want_to_read_some_config_files_from_the_same_place.)
29. [How can I display the target of a symbolic link?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F029.How_can_I_display_the_target_of_a_symbolic_link.3F)
30. [How can I rename all my *.foo files to *.bar, or convert spaces to underscores, or convert upper-case file names to lower case?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F030.How_can_I_rename_all_my_.2A.foo_files_to_.2A.bar.2C_or_convert_spaces_to_underscores.2C_or_convert_upper-case_file_names_to_lower_case.3F)
31. [What is the difference between test, [ and [[ ?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F031.What_is_the_difference_between_test.2C_.5B_and_.5B.5B_.3F)
32. [How can I redirect the output of 'time' to a variable or file?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F032.How_can_I_redirect_the_output_of_.27time.27_to_a_variable_or_file.3F)
33. [How can I find a process ID for a process given its name?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F033.How_can_I_find_a_process_ID_for_a_process_given_its_name.3F)
34. [Can I do a spinner in Bash?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F034.Can_I_do_a_spinner_in_Bash.3F)
35. [How can I handle command-line options and arguments in my script easily?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F035.How_can_I_handle_command-line_options_and_arguments_in_my_script_easily.3F)
36. [How can I get all lines that are: in both of two files (set intersection) or in only one of two files (set subtraction).](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F036.How_can_I_get_all_lines_that_are:_in_both_of_two_files_.28set_intersection.29_or_in_only_one_of_two_files_.28set_subtraction.29.)
37. [How can I print text in various colors?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F037.How_can_I_print_text_in_various_colors.3F)
38. [How do Unix file permissions work?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F038.How_do_Unix_file_permissions_work.3F)
39. [What are all the dot-files that bash reads?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F039.What_are_all_the_dot-files_that_bash_reads.3F)
40. [How do I use dialog to get input from the user?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F040.How_do_I_use_dialog_to_get_input_from_the_user.3F)
41. [How do I determine whether a variable contains a substring?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F041.How_do_I_determine_whether_a_variable_contains_a_substring.3F)
42. [How can I find out if a process is still running?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F042.How_can_I_find_out_if_a_process_is_still_running.3F)
43. [Why does my crontab job fail? 0 0 * * * some command > /var/log/mylog.`date +%Y%m%d`](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F043.Why_does_my_crontab_job_fail.3F__0_0_.2A_.2A_.2A_some_command_.3E_.2Fvar.2Flog.2Fmylog..60date_.2B-.25Y.25m.25d.60)
44. [How do I create a progress bar? How do I see a progress indicator when copying/moving files?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F044.How_do_I_create_a_progress_bar.3F__How_do_I_see_a_progress_indicator_when_copying.2Fmoving_files.3F)
45. [How can I ensure that only one instance of a script is running at a time (mutual exclusion, locking)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F045.How_can_I_ensure_that_only_one_instance_of_a_script_is_running_at_a_time_.28mutual_exclusion.2C_locking.29.3F)
46. [I want to check to see whether a word is in a list (or an element is a member of a set).](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F046.I_want_to_check_to_see_whether_a_word_is_in_a_list_.28or_an_element_is_a_member_of_a_set.29.)
47. [How can I redirect stderr to a pipe?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F047.How_can_I_redirect_stderr_to_a_pipe.3F)
48. [Eval command and security issues](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F048.Eval_command_and_security_issues)
49. [How can I view periodic updates/appends to a file? (ex: growing log file)](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F049.How_can_I_view_periodic_updates.2Fappends_to_a_file.3F_.28ex:_growing_log_file.29)
50. [I'm trying to put a command in a variable, but the complex cases always fail!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F050.I.27m_trying_to_put_a_command_in_a_variable.2C_but_the_complex_cases_always_fail.21)
51. [I want history-search just like in tcsh. How can I bind it to the up and down keys?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F051.I_want_history-search_just_like_in_tcsh._How_can_I_bind_it_to_the_up_and_down_keys.3F)
52. [How do I convert a file from DOS format to UNIX format (remove CRs from CR-LF line terminators)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F052.How_do_I_convert_a_file_from_DOS_format_to_UNIX_format_.28remove_CRs_from_CR-LF_line_terminators.29.3F)
53. [I have a fancy prompt with colors, and now bash doesn't seem to know how wide my terminal is. Lines wrap around incorrectly.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F053.I_have_a_fancy_prompt_with_colors.2C_and_now_bash_doesn.27t_seem_to_know_how_wide_my_terminal_is.__Lines_wrap_around_incorrectly.)
    1. [Escape the colors with \[ \]](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F053.Escape_the_colors_with_.2BAFw.5B_.2BAFw.5D)
54. [How can I tell whether a variable contains a valid number?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F054.How_can_I_tell_whether_a_variable_contains_a_valid_number.3F)
55. [Tell me all about 2>&1 -- what's the difference between 2>&1 >foo and >foo 2>&1, and when do I use which?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F055.Tell_me_all_about_2.3E.261_--_what.27s_the_difference_between_2.3E.261_.3Efoo_and_.3Efoo_2.3E.261.2C_and_when_do_I_use_which.3F)
56. [How can I untar (or unzip) multiple tarballs at once?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F056.How_can_I_untar_.28or_unzip.29_multiple_tarballs_at_once.3F)
57. [How can I group entries (in a file by common prefixes)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F057.How_can_I_group_entries_.28in_a_file_by_common_prefixes.29.3F)
58. [Can bash handle binary data?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F058.Can_bash_handle_binary_data.3F)
59. [I saw this command somewhere: :(){ :|:& } (fork bomb). How does it work?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F059.I_saw_this_command_somewhere:_:.28.29.7B_:.7C:.26_.7D_.28fork_bomb.29.__How_does_it_work.3F)
60. [I'm trying to write a script that will change directory (or set a variable), but after the script finishes, I'm back where I started (or my variable isn't set)!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F060.I.27m_trying_to_write_a_script_that_will_change_directory_.28or_set_a_variable.29.2C_but_after_the_script_finishes.2C_I.27m_back_where_I_started_.28or_my_variable_isn.27t_set.29.21)
61. [Is there a list of which features were added to specific releases (versions) of Bash?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F061.Is_there_a_list_of_which_features_were_added_to_specific_releases_.28versions.29_of_Bash.3F)
62. [How do I create a temporary file in a secure manner?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F062.How_do_I_create_a_temporary_file_in_a_secure_manner.3F)
63. [My ssh client hangs when I try to logout after running a remote background job!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F063.My_ssh_client_hangs_when_I_try_to_logout_after_running_a_remote_background_job.21)
64. [Why is it so hard to get an answer to the question that I asked in #bash?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F064.Why_is_it_so_hard_to_get_an_answer_to_the_question_that_I_asked_in_.23bash.3F)
65. [Is there a "PAUSE" command in bash like there is in MSDOS batch scripts? To prompt the user to press any key to continue?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F065.Is_there_a_.22PAUSE.22_command_in_bash_like_there_is_in_MSDOS_batch_scripts.3F__To_prompt_the_user_to_press_any_key_to_continue.3F)
66. [I want to check if [[ $var == foo || $var == bar || $var == more ]] without repeating $var n times.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F066.I_want_to_check_if_.5B.5B_.24var_.3D.3D_foo_.7C.7C_.24var_.3D.3D_bar_.7C.7C_.24var_.3D.3D_more_.5D.5D_without_repeating_.24var_n_times.)
67. [How can I trim leading/trailing white space from one of my variables?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F067.How_can_I_trim_leading.2Ftrailing_white_space_from_one_of_my_variables.3F)
68. [How do I run a command, and have it abort (timeout) after N seconds?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F068.How_do_I_run_a_command.2C_and_have_it_abort_.28timeout.29_after_N_seconds.3F)
69. [I want to automate an ssh (or scp, or sftp) connection, but I don't know how to send the password....](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F069.I_want_to_automate_an_ssh_.28or_scp.2C_or_sftp.29_connection.2C_but_I_don.27t_know_how_to_send_the_password....)
70. [How do I convert Unix (epoch) times to human-readable values?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F070.How_do_I_convert_Unix_.28epoch.29_times_to_human-readable_values.3F)
71. [How do I convert an ASCII character to its decimal (or hexadecimal) value and back? How do I do URL encoding or URL decoding?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F071.How_do_I_convert_an_ASCII_character_to_its_decimal_.28or_hexadecimal.29_value_and_back.3F_How_do_I_do_URL_encoding_or_URL_decoding.3F)
72. [How can I ensure my environment is configured for cron, batch, and at jobs?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F072.How_can_I_ensure_my_environment_is_configured_for_cron.2C_batch.2C_and_at_jobs.3F)
73. [How can I use parameter expansion? How can I get substrings? How can I get a file without its extension, or get just a file's extension? What are some good ways to do basename and dirname?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F073.How_can_I_use_parameter_expansion.3F__How_can_I_get_substrings.3F__How_can_I_get_a_file_without_its_extension.2C_or_get_just_a_file.27s_extension.3F_What_are_some_good_ways_to_do_basename_and_dirname.3F)
74. [How do I get the effects of those nifty Bash Parameter Expansions in older shells?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F074.How_do_I_get_the_effects_of_those_nifty_Bash_Parameter_Expansions_in_older_shells.3F)
75. [How do I use 'find'? I can't understand the man page at all!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F075.How_do_I_use_.27find.27.3F__I_can.27t_understand_the_man_page_at_all.21)
76. [How do I get the sum of all the numbers in a column?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F076.How_do_I_get_the_sum_of_all_the_numbers_in_a_column.3F)
77. [How do I log history or "secure" bash against history removal?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F077.How_do_I_log_history_or_.22secure.22_bash_against_history_removal.3F)
78. [I want to set a user's password using the Unix passwd command, but how do I script that? It doesn't read standard input!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F078.I_want_to_set_a_user.27s_password_using_the_Unix_passwd_command.2C_but_how_do_I_script_that.3F__It_doesn.27t_read_standard_input.21)
79. [How can I grep for lines containing foo AND bar, foo OR bar? Or for files containing foo AND bar, possibly on separate lines? Or files containing foo but NOT bar?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F079.How_can_I_grep_for_lines_containing_foo_AND_bar.2C_foo_OR_bar.3F__Or_for_files_containing_foo_AND_bar.2C_possibly_on_separate_lines.3F__Or_files_containing_foo_but_NOT_bar.3F)
80. [How can I make an alias that takes an argument?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F080.How_can_I_make_an_alias_that_takes_an_argument.3F)
81. [How can I determine whether a command exists anywhere in my PATH?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F081.How_can_I_determine_whether_a_command_exists_anywhere_in_my_PATH.3F)
82. [Why is $(...) preferred over `...` (backticks)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F082.Why_is_.24.28....29_preferred_over_.60....60_.28backticks.29.3F)
83. [How do I determine whether a variable is already defined? Or a function?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F083.How_do_I_determine_whether_a_variable_is_already_defined.3F__Or_a_function.3F)
84. [How do I return a string (or large number, or negative number) from a function? "return" only lets me give a number from 0 to 255.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F084.How_do_I_return_a_string_.28or_large_number.2C_or_negative_number.29_from_a_function.3F__.22return.22_only_lets_me_give_a_number_from_0_to_255.)
85. [How to write several times to a fifo without having to reopen it?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F085.How_to_write_several_times_to_a_fifo_without_having_to_reopen_it.3F)
86. [How to ignore aliases, functions, or builtins when running a command?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F086.How_to_ignore_aliases.2C_functions.2C_or_builtins_when_running_a_command.3F)
87. [How can I get a file's permissions (or other metadata) without parsing ls -l output?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F087.How_can_I_get_a_file.27s_permissions_.28or_other_metadata.29_without_parsing_ls_-l_output.3F)
88. [How can I avoid losing any history lines?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F088.How_can_I_avoid_losing_any_history_lines.3F)
89. [I'm reading a file line by line and running ssh or ffmpeg, only the first line gets processed!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F089.I.27m_reading_a_file_line_by_line_and_running_ssh_or_ffmpeg.2C_only_the_first_line_gets_processed.21)
90. [How do I prepend a text to a file (the opposite of >>)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F090.How_do_I_prepend_a_text_to_a_file_.28the_opposite_of_.3E.3E.29.3F)
91. [I'm trying to get the number of columns or lines of my terminal but the variables COLUMNS / LINES are always empty.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F091.I.27m_trying_to_get_the_number_of_columns_or_lines_of_my_terminal_but_the_variables_COLUMNS_.2F_LINES_are_always_empty.)
92. [How do I write a CGI script that accepts parameters?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F092.How_do_I_write_a_CGI_script_that_accepts_parameters.3F)
93. [How can I set the contents of my terminal's title bar?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F093.How_can_I_set_the_contents_of_my_terminal.27s_title_bar.3F)
94. [I want to get an alert when my disk is full (parsing df output).](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F094.I_want_to_get_an_alert_when_my_disk_is_full_.28parsing_df_output.29.)
95. [I'm getting "Argument list too long". How can I process a large list in chunks?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F095.I.27m_getting_.22Argument_list_too_long.22.__How_can_I_process_a_large_list_in_chunks.3F)
96. [ssh eats my word boundaries! I can't do ssh remotehost make CFLAGS="-g -O"!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F096.ssh_eats_my_word_boundaries.21__I_can.27t_do_ssh_remotehost_make_CFLAGS.3D.22-g_-O.22.21)
97. [How do I determine whether a symlink is dangling (broken)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F097.How_do_I_determine_whether_a_symlink_is_dangling_.28broken.29.3F)
98. [How to add localization support to your bash scripts](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F098.How_to_add_localization_support_to_your_bash_scripts)
99. [How can I get the newest (or oldest) file from a directory?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F099.How_can_I_get_the_newest_.28or_oldest.29_file_from_a_directory.3F)
100. [How do I do string manipulations in bash?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F100.How_do_I_do_string_manipulations_in_bash.3F)
101. [Common utility functions (warn, die)](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F101.Common_utility_functions_.28warn.2C_die.29)
102. [How to get the difference between two dates](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F102.How_to_get_the_difference_between_two_dates)
103. [How do I check whether my file was modified in a certain month or date range?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F103.How_do_I_check_whether_my_file_was_modified_in_a_certain_month_or_date_range.3F)
104. [Why doesn't foo=bar echo "$foo" print bar?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F104.Why_doesn.27t_foo.3Dbar_echo_.22.24foo.22_print_bar.3F)
105. [Why doesn't set -e (or set -o errexit, or trap ERR) do what I expected?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F105.Why_doesn.27t_set_-e_.28or_set_-o_errexit.2C_or_trap_ERR.29_do_what_I_expected.3F)
106. [Logging! I want to send all of my script's output to a log file. But I want to do it from inside the script. And I want to see it on the terminal too!](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F106.Logging.21_I_want_to_send_all_of_my_script.27s_output_to_a_log_file._But_I_want_to_do_it_from_inside_the_script._And_I_want_to_see_it_on_the_terminal_too.21)
107. [How do I add a timestamp to every line of a stream?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F107.How_do_I_add_a_timestamp_to_every_line_of_a_stream.3F)
108. [How do I wait for several spawned processes?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F108.How_do_I_wait_for_several_spawned_processes.3F)
109. [How can I tell whether my script was sourced (dotted in) or executed?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F109.How_can_I_tell_whether_my_script_was_sourced_.28dotted_in.29_or_executed.3F)
110. [How do I copy a file to a remote system, and specify a remote name which may contain spaces?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F110.How_do_I_copy_a_file_to_a_remote_system.2C_and_specify_a_remote_name_which_may_contain_spaces.3F)
111. [What is the Shellshock vulnerability in Bash?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F111.What_is_the_Shellshock_vulnerability_in_Bash.3F)
112. [What are the advantages and disadvantages of using set -u (or set -o nounset)?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F112.What_are_the_advantages_and_disadvantages_of_using_set_-u_.28or_set_-o_nounset.29.3F)
113. [How do I extract data from an HTML or XML file?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F113.How_do_I_extract_data_from_an_HTML_or_XML_file.3F)
114. [How do I operate on IP addresses and netmasks?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F114.How_do_I_operate_on_IP_addresses_and_netmasks.3F)
115. [How do I make a menu?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F115.How_do_I_make_a_menu.3F)
116. [I have two files. The first one contains bad IP addresses (plus other fields). I want to remove all of these bad addresses from a second file.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F116.I_have_two_files.__The_first_one_contains_bad_IP_addresses_.28plus_other_fields.29.__I_want_to_remove_all_of_these_bad_addresses_from_a_second_file.)
117. [I have a pipeline where a long-running command feeds into a filter. If the filter finds "foo", I want the long-running command to die.](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F117.I_have_a_pipeline_where_a_long-running_command_feeds_into_a_filter._If_the_filter_finds_.22foo.22.2C_I_want_the_long-running_command_to_die.)
118. [How do I print the contents of an array in reverse order, or reverse an array?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F118.How_do_I_print_the_contents_of_an_array_in_reverse_order.2C_or_reverse_an_array.3F)
119. [What's the difference between "cmd < file" and "cat file | cmd"? What is a UUOC?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F119.What.27s_the_difference_between_.22cmd_.3C_file.22_and_.22cat_file_.7C_cmd.22.3F__What_is_a_UUOC.3F)
120. [How can I find out where this strange variable in my interactive shell came from?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F120.How_can_I_find_out_where_this_strange_variable_in_my_interactive_shell_came_from.3F)
121. [What does value too great for base mean? (Octal values in arithmetic.)](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F121.What_does_value_too_great_for_base_mean.3F_.28Octal_values_in_arithmetic..29)
122. [How do I print a horizontal line of characters like ----?](https://mywiki.wooledge.org/BashFAQ#BashFAQ.2F122.How_do_I_print_a_horizontal_line_of_characters_like_----.3F)

## 1. How can I read a file (data stream, variable) line-by-line (and/or field-by-field)?

[Don't try to use "for"](https://mywiki.wooledge.org/DontReadLinesWithFor). Use a while loop and the read command. Here is the basic template; there are many variations to discuss:

## 2. How can I store the return value and/or output of a command in a variable?

Well, that depends on whether you want to store the command's _output_ (either stdout, or stdout + stderr) or its _exit status_ (0 to 255, with 0 typically meaning "success").

The tempting solution is to use ls to output sorted filenames and operate on the results using e.g. awk. As usual, the ls approach [cannot be made robust](https://mywiki.wooledge.org/ParsingLs) and should never be used in scripts due in part to the possibility of arbitrary characters (including newlines) present in filenames. Therefore, we need some other way to compare file metadata.

## 4. How can I check whether a directory is empty or not? How do I check for any *.mpg files, or count how many there are?

In Bash, you can count files safely and easily with the nullglob and dotglob options (which change the behaviour of [globbing](https://mywiki.wooledge.org/glob)), and an [array](https://mywiki.wooledge.org/BashFAQ/005):

## 5. How can I use array variables?

This answer assumes you have a basic understanding of what arrays _are_. If you're new to this kind of programming, you may wish to start with [the guide's explanation](https://mywiki.wooledge.org/BashGuide/Arrays). This page is more thorough. See [links](https://mywiki.wooledge.org/BashFAQ/005#See_Also) at the bottom for more resources.

## 6. How can I use variable variables (indirect variables, pointers, references) or associative arrays?

This is a complex page, because it's a complex topic. It's been divided into roughly four parts: associative arrays, name references, evaluating indirect variables, and assigning indirect variables. There are discussions of programming issues and concepts scattered throughout.

## 7. Is there a function to return the length of a string?

The fastest way, not requiring external programs (but not usable in Bourne shells):

## 8. How can I recursively search all files for a string?

If you are on a typical GNU or BSD system, all you need is one of these:

## 9. What is buffering? Or, why does my command line produce no output: tail -f logfile | grep 'foo bar' | awk ...

Most standard Unix commands buffer their output when used non-interactively. This means that they don't write each character (or even each line) immediately, but instead collect a larger number of characters (often 4 kilobytes) before printing anything at all. In the case above, the grep command buffers its output, and therefore awk only gets its input in large chunks.

## 10. How can I recreate a directory hierarchy structure, without the files?

## 11. How can I print the n'th line of a file?

One dirty (but not quick) way is:

## 12. How do I invoke a shell command from a non-shell application?

You can use the shell's -c option to run the shell with the sole purpose of executing a short bit of script:

## 13. How can I concatenate two variables? How do I append a string to a variable?

There is no (explicit) concatenation operator for strings (either literal or variable dereferences) in the shell; you just write them adjacent to each other:

## 14. How can I redirect the output of multiple commands at once?

Redirecting the standard output of a single command is as easy as:

## 15. How can I run a command on all files with the extension .gz?

Often a command already accepts several files as arguments, e.g.

## 16. How can I use a logical AND/OR/NOT in a shell pattern (glob)?

["Globs"](https://mywiki.wooledge.org/glob) are simple patterns that can be used to match filenames or strings. They're generally not very powerful. If you need more power, there are a few options available.

## 17. How can I group expressions in an if statement, e.g. if (A AND B) OR C?

The portable (POSIX or Bourne) way is to use multiple test (or [) commands:

## 18. How can I use numbers with leading zeros in a loop, e.g. 01, 02?

As always, there are many different ways to solve the problem, each with its own advantages and disadvantages. The most important considerations are which shell you're using, whether the start/end numbers are constants, and how many times the loop is going to iterate.

## 19. How can I split a file into line ranges, e.g. lines 1-10, 11-20, 21-30?

POSIX specifies the split utility, which can be used for this purpose:

## 20. How can I find and safely handle file names containing newlines, spaces or both?

First and foremost, to understand why you're having trouble, read [Arguments](https://mywiki.wooledge.org/Arguments) to get a grasp on how the shell understands the statements you give it. It is vital that you grasp this matter well if you're going to be doing anything with the shell.

## 21. How can I replace a string with another string in a variable, a stream, a file, or in all the files in a directory?

There are a number of techniques for this. Which one to use depends on many factors, the biggest of which is _what we're editing_. This page also contains contradictory advice from multiple authors. This is a deeply _ugly_ topic, and there are no universally right answers (but plenty of universally _wrong_ ones).

## 22. How can I calculate with floating point numbers instead of just integers?

[BASH](https://mywiki.wooledge.org/BASH)'s builtin [arithmetic](https://mywiki.wooledge.org/ArithmeticExpression) uses integers only:

$ printf '%s\n' "$((10 / 3))"
3

## 23. I want to launch an interactive shell that has special aliases and functions, not the ones in the user's ~/.bashrc.

When starting bash in non-POSIX mode, specify a different start-up file with --rcfile:

## 24. I set variables in a loop that's in a pipeline. Why do they disappear after the loop terminates? Or, why can't I pipe data to read?

In most shells, each command of a pipeline is executed in a separate [SubShell](https://mywiki.wooledge.org/SubShell). Non-working example:

## 25. How can I access positional parameters after $9?

Use ${10} instead of $10. This works for [BASH](https://mywiki.wooledge.org/BASH) and [KornShell](https://mywiki.wooledge.org/KornShell) and is standardized by [POSIX](https://mywiki.wooledge.org/POSIX), but it doesn't work for older [BourneShell](https://mywiki.wooledge.org/BourneShell) implementations. Another way to access arbitrary positional parameters after $9 is to use for, e.g. to get the last parameter:

## 26. How can I randomize (shuffle) the order of lines in a file? Or select a random line from a file, or select a random file from a directory?

To randomize the lines of a file, here is one approach. This one involves generating a random number, which is prefixed to each line; then sorting the resulting lines, and removing the numbers.

Two unrelated processes cannot use the arguments, the environment or stdin/stdout to communicate; some form of inter-process communication (IPC) is required.

## 28. How do I determine the location of my script? I want to read some config files from the same place.

There are two prime reasons why this issue comes up: either you want to externalize data or configuration of your script and need a way to find these external resources, or your script is intended to act upon a bundle of some sort (e.g., a build script) and needs to find the resources to act upon.

## 29. How can I display the target of a symbolic link?

The nonstandard external command readlink(1) can be used to display the target of a symbolic link:

## 30. How can I rename all my *.foo files to *.bar, or convert spaces to underscores, or convert upper-case file names to lower case?

There are a bunch of different ways to do this, depending on which nonstandard tools you have available. Even with just standard POSIX tools, you can still perform most of the simple cases. We'll show the portable tool examples first.

## 31. What is the difference between test, [ and [[ ?

The open square bracket [ command (aka test command) and the [[ ... ]] test construct are used to evaluate expressions. [[ ... ]] works only in the Korn shell (where it originates), Bash, Zsh, and recent versions of Yash and busybox sh (if enabled at compilation time, and still very limited there especially in the hush-based variant), and is more powerful; [ and test are POSIX utilities (generally builtin). POSIX doesn't specify the [[ ... ]] construct (which has a specific syntax with significant variations between implementations) though allows shells to treat [[ as a keyword. Here are some examples:

## 32. How can I redirect the output of 'time' to a variable or file?

Bash's time keyword uses special trickery, so that you can do things like

## 33. How can I find a process ID for a process given its name?

Usually a process is referred to using its process ID (PID), and the ps(1) command can display the information for any process given its process ID, e.g.

## 34. Can I do a spinner in Bash?

Sure!

## 35. How can I handle command-line options and arguments in my script easily?

Well, that depends a great deal on what you want to do with them. There are two standard approaches, each with its strengths and weaknesses.

## 36. How can I get all lines that are: in both of two files (set intersection) or in only one of two files (set subtraction).

Use the comm(1) command:

## 37. How can I print text in various colors?

_Do not_ hard-code ANSI color escape sequences in your program! The tput command lets you interact with the terminal database in a sane way:

## 38. How do Unix file permissions work?

See [Permissions](https://mywiki.wooledge.org/Permissions).

## 39. What are all the dot-files that bash reads?

See [DotFiles](https://mywiki.wooledge.org/DotFiles).

## 40. How do I use dialog to get input from the user?

Here is an example:

## 41. How do I determine whether a variable contains a substring?

There are many choices here: you can perform an exact substring match, or a [glob](https://mywiki.wooledge.org/glob)-style pattern match, or a [RegularExpression](https://mywiki.wooledge.org/RegularExpression) match.

## 42. How can I find out if a process is still running?

The kill command is used to send signals to a running process. As a convenience function, the signal "0", which does not exist, can be used to find out if a process is still running:

## 43. Why does my crontab job fail? 0 0 * * * some command > /var/log/mylog.`date +%Y%m%d`

In many versions of crontab, the percent sign (%) is treated specially, and therefore must be escaped with backslashes:

## 44. How do I create a progress bar? How do I see a progress indicator when copying/moving files?

The easiest way to add a progress bar to your own script is to use dialog --gauge. Here is an example, which relies heavily on [BASH](https://mywiki.wooledge.org/BASH) features:

## 45. How can I ensure that only one instance of a script is running at a time (mutual exclusion, locking)?

We need some means of _mutual exclusion_. One way is to use a "lock": any number of processes can try to acquire the lock simultaneously, but only one of them will succeed.

## 46. I want to check to see whether a word is in a list (or an element is a member of a set).

If your real question is _How do I check whether one of my parameters was -v?_ then see [FAQ #35](https://mywiki.wooledge.org/BashFAQ/035) instead. Otherwise, read on…

## 47. How can I redirect stderr to a pipe?

A pipe can only carry standard output (stdout) of a program. To pipe standard error (stderr) through it, you need to redirect stderr to the same destination as stdout. Optionally you can close stdout or redirect it to /dev/null to only get stderr. Some sample code:

## 48. Eval command and security issues

The eval command is extremely powerful and extremely easy to abuse.

## 49. How can I view periodic updates/appends to a file? (ex: growing log file)

tail -f will show you the growing log file. On some systems (e.g. OpenBSD), this will automatically track a rotated log file to the new file with the same name (which is usually what you want). To get the equivalent functionality on GNU systems, use tail -F instead.

## 50. I'm trying to put a command in a variable, but the complex cases always fail!

Variables hold data. Functions hold code. Don't put code inside variables! There are many situations in which people try to shove commands, or command arguments, into variables and then run them. Each case needs to be handled separately.

## 51. I want history-search just like in tcsh. How can I bind it to the up and down keys?

Just add the following to /etc/inputrc or your ~/.inputrc:

## 52. How do I convert a file from DOS format to UNIX format (remove CRs from CR-LF line terminators)?

Carriage return (CR) characters are used in line ending markers on some systems. There are three different kinds of line endings in common use:

## 53. I have a fancy prompt with colors, and now bash doesn't seem to know how wide my terminal is. Lines wrap around incorrectly.

### 53.1. Escape the colors with \[ \]

You must put \[ and \] around any non-printing escape sequences in your prompt. Thus:

## 54. How can I tell whether a variable contains a valid number?

First, you have to define what you mean by "number". The most common case when people ask this seems to be "a non-negative integer, with no leading + sign". Or in other words, a string of all digits. Other times, people want to validate a floating-point input, with optional sign and optional decimal point.

## 55. Tell me all about 2>&1 -- what's the difference between 2>&1 >foo and >foo 2>&1, and when do I use which?

Bash processes all [redirections](https://mywiki.wooledge.org/Redirection) from left to right, in order. And the order is significant. Moving them around within a command may change the results of that command.

## 56. How can I untar (or unzip) multiple tarballs at once?

As the tar command was originally designed to read from and write to tape devices (tar - Tape ARchiver), you can specify only filenames to put inside an archive (write to tape) or to extract out of an archive (read from tape).

## 57. How can I group entries (in a file by common prefixes)?

As in, one wants to convert:

## 58. Can bash handle binary data?

The answer is, basically, no....

## 59. I saw this command somewhere: :(){ :|:& } (fork bomb). How does it work?

**This is a potentially dangerous command. Don't run it!** The "trigger" is omitted from the question above, leaving only the part that sets up the function.

## 60. I'm trying to write a script that will change directory (or set a variable), but after the script finishes, I'm back where I started (or my variable isn't set)!

Consider this:

   #!/bin/sh
   cd /tmp

## 61. Is there a list of which features were added to specific releases (versions) of Bash?

Here are some links to official Bash documentation:

## 62. How do I create a temporary file in a secure manner?

This question is distressingly difficult to answer, because there isn't any single command that simply _works_ everywhere. This page will discuss the difficulties involved, and will offer a variety of solutions. Which one you choose will depend on your script's needs and expected runtime environment.

## 63. My ssh client hangs when I try to logout after running a remote background job!

The following will not do what you expect:

   ssh me@remotehost 'sleep 120 &'
   # Client hangs for 120 seconds

## 64. Why is it so hard to get an answer to the question that I asked in #bash?

Maybe nobody knows the answer (or the people who know the answer are busy). Maybe you haven't given enough detail about the problem, or you [haven't presented the problem clearly](https://mywiki.wooledge.org/BadQuestions). Maybe the question you asked is answered in this FAQ, or in [BashPitfalls](https://mywiki.wooledge.org/BashPitfalls), or in the [BashGuide](https://mywiki.wooledge.org/BashGuide).

## 65. Is there a "PAUSE" command in bash like there is in MSDOS batch scripts? To prompt the user to press any key to continue?

Use the following to wait until the user presses enter:

## 66. I want to check if [[ $var == foo || $var == bar || $var == more ]] without repeating $var n times.

The portable solution uses case:

## 67. How can I trim leading/trailing white space from one of my variables?

There are a few ways to do this. Some involve special tricks that only work with whitespace. Others are more general, and can be used to strip leading zeroes, etc.

## 68. How do I run a command, and have it abort (timeout) after N seconds?

**FIRST** check whether the command you're running can be told to timeout directly. The methods described here are "hacky" workarounds to force a command to terminate after a certain time has elapsed. Configuring your command properly is _always_ preferable to the alternatives below.

## 69. I want to automate an ssh (or scp, or sftp) connection, but I don't know how to send the password....

When dealing with authentication in a shell script, please bear in mind the following points:

## 70. How do I convert Unix (epoch) times to human-readable values?

The only sane way to handle time values within a program is to convert them into a linear scale. You can't store "January 17, 2005 at 5:37 PM" in a variable and expect to do anything with it....

## 71. How do I convert an ASCII character to its decimal (or hexadecimal) value and back? How do I do URL encoding or URL decoding?

If you have a known octal or hexadecimal value (at script-writing time), you can just use printf:

## 72. How can I ensure my environment is configured for cron, batch, and at jobs?

If a shell or other script calling shell commands runs fine interactively but fails due to environment configurations (say: a complex $PATH) when run noninteractively, you'll need to force your environment to be properly configured.

## 73. How can I use parameter expansion? How can I get substrings? How can I get a file without its extension, or get just a file's extension? What are some good ways to do basename and dirname?

Parameter expansion is an important subject. This page contains a concise overview of parameter expansion.

## 74. How do I get the effects of those nifty Bash Parameter Expansions in older shells?

Most of the extended forms of [parameter expansion](https://mywiki.wooledge.org/BashFAQ/073) do not work with the older [BourneShell](https://mywiki.wooledge.org/BourneShell). If your code needs to be portable to that shell as well, sed and expr can often be used.

## 75. How do I use 'find'? I can't understand the man page at all!

See [UsingFind](https://mywiki.wooledge.org/UsingFind).

## 76. How do I get the sum of all the numbers in a column?

This and all similar questions are best answered with an [AWK](https://mywiki.wooledge.org/AWK) one-liner.

## 77. How do I log history or "secure" bash against history removal?

If you're a shell user who wants to record your own activities, see [FAQ #88](https://mywiki.wooledge.org/BashFAQ/088) instead. If you're a system administrator who wants to know how to find out what a user had executed when they unset or /dev/nulled their shell history, there are several problems with this....

## 78. I want to set a user's password using the Unix passwd command, but how do I script that? It doesn't read standard input!

OK, first of all, I _know_ there are going to be some people reading this, right now, who don't even understand the question. Here, this **does not work**:

{ echo oldpass; echo newpass; echo newpass; } | passwd
# This DOES NOT WORK!

## 79. How can I grep for lines containing foo AND bar, foo OR bar? Or for files containing foo AND bar, possibly on separate lines? Or files containing foo but NOT bar?

This is really four different questions, so we'll break this answer into parts.

## 80. How can I make an alias that takes an argument?

You can't. Aliases in bash are extremely rudimentary, and not really suitable to any serious purpose. The bash man page even says so explicitly:

## 81. How can I determine whether a command exists anywhere in my PATH?

POSIX specifies a shell builtins called command and type which can be used for this purpose. Note that type's exit codes isn't well defined by POSIX whereas command's exit status is well defined by POSIX, so that one is probably the safest to use.

## 82. Why is $(...) preferred over `...` (backticks)?

The `cmd` backtick format is the legacy syntax for [command substitution](https://mywiki.wooledge.org/CommandSubstitution), required only by more than 30-year-old [Bourne shells](https://mywiki.wooledge.org/BourneShell). The modern POSIX $(...) syntax is preferred for many reasons:

## 83. How do I determine whether a variable is already defined? Or a function?

There are several ways to test these things, depending on the exact requirements. Most of the time, the desired test is _whether a variable has a non-empty value_. In this case, we may simply use:

## 84. How do I return a string (or large number, or negative number) from a function? "return" only lets me give a number from 0 to 255.

Functions in Bash (as well as all the other Bourne-family shells) work like commands: that is, they only "return" an exit status, which is an integer from 0 to 255 inclusive. This is intended to be used only for signaling errors, not for returning the results of computations, or other data.

## 85. How to write several times to a fifo without having to reopen it?

In the general case, you'll open a new [FileDescriptor](https://mywiki.wooledge.org/FileDescriptor) (FD) pointing to the fifo, and write through that. For simple cases, it may be possible to skip that step.

## 86. How to ignore aliases, functions, or builtins when running a command?

functions, builtins, external utilities, and aliases can all be defined with the same name at once. It's sometimes necessary specify which of these the shell should resolve while bypassing the others.

There are several potential ways, most of which are system-specific. They also depend on precisely _why_ you want the information; in most cases, there will be some other way to accomplish your [real goal](https://mywiki.wooledge.org/XyProblem). You [don't want to parse ls's output](https://mywiki.wooledge.org/ParsingLs) if there's any possible way to avoid doing so.

## 88. How can I avoid losing any history lines?

## 89. I'm reading a file line by line and running ssh or ffmpeg, only the first line gets processed!

When [reading a file line by line](https://mywiki.wooledge.org/BashFAQ/001), if a command inside the loop also reads stdin, it can exhaust the input file. For example:

## 90. How do I prepend a text to a file (the opposite of >>)?

You cannot do it with bash redirections alone; the opposite of >> does not exist....

## 91. I'm trying to get the number of columns or lines of my terminal but the variables COLUMNS / LINES are always empty.

COLUMNS and LINES are set by [BASH](https://mywiki.wooledge.org/BASH) in interactive mode; they are not available by default in a script. On most systems, you can try to query the terminal yourself:

## 92. How do I write a CGI script that accepts parameters?

There are always circumstances beyond our control that drive us to do things that we would never choose to do on our own. This FAQ entry describes one of those situations.

## 93. How can I set the contents of my terminal's title bar?

If you have a terminal that understands xterm-compatible escape sequences, and you just want to set the title one time, you can use a function like this:

## 94. I want to get an alert when my disk is full (parsing df output).

Sadly, parsing the output of df really is the most reliable way to determine how full a disk is, on most operating systems. However, please note that this is a "least bad" answer, not a "best" answer. Parsing any command-line reporting tool's output in a program is never pretty. The purpose of this FAQ is to try to describe all the problems this approach is known to encounter, and work around them.

## 95. I'm getting "Argument list too long". How can I process a large list in chunks?

First, let's review some background material. When a process wants to run another process, it fork()s a child, and the child calls one of the exec* family of system calls (e.g. execve()), giving the name or path of the new process's program file; the name of the new process; the list of arguments for the new process; and, in some cases, a set of environment variables. Thus:

## 96. ssh eats my word boundaries! I can't do ssh remotehost make CFLAGS="-g -O"!

[ssh](http://www.openssh.org/) emulates the behavior of the Unix remote shell command (rsh or remsh), including this bug. There are a few ways to work around it, depending on exactly what you need.

## 97. How do I determine whether a symlink is dangling (broken)?

The documentation on this is fuzzy, but it turns out you _can_ do this with shell builtins:

## 98. How to add localization support to your bash scripts

Looking for examples of how to add simple localization to your bash scripts, and how to do testing? This is probably what you need....

## 99. How can I get the newest (or oldest) file from a directory?

This page should be merged with [BashFAQ/003](https://mywiki.wooledge.org/BashFAQ/003)

## 100. How do I do string manipulations in bash?

Bash can do string operations. LOTS of string operations. This is an introduction to bash string manipulations and related techniques. It overlaps with the [Parameter Expansion](https://mywiki.wooledge.org/BashFAQ/073) question, but the information here is presented in a more beginner-friendly manner (we hope).

## 101. Common utility functions (warn, die)

(If you were looking for option processing, see [BashFAQ/035](https://mywiki.wooledge.org/BashFAQ/035).) Bash and sh don't offer a die builtin command like Perl does, but it's common to use a die function in scripts. You just have to write one yourself. Most people who write a die function like to keep it simple. There are two common varieties: one that only takes a message to print, and one that takes a message and an exit status value.

## 102. How to get the difference between two dates

It's best if you work with timestamps throughout your code, and then only convert timestamps to human-readable formats for output. If you must handle human-readable dates as input, then you will need something that can parse them.

## 103. How do I check whether my file was modified in a certain month or date range?

Doing date-related math in Bash is hard because Bash has no builtins in place for doing math with dates or getting metadata such as modification time from files.

## 104. Why doesn't foo=bar echo "$foo" print bar?

This is subtle, and has to do with the exact order in which the [BashParser](https://mywiki.wooledge.org/BashParser) performs each step.

## 105. Why doesn't set -e (or set -o errexit, or trap ERR) do what I expected?

set -e was an attempt to add "automatic error detection" to the shell. Its goal was to cause the shell to abort any time an error occurred, so you don't have to put || exit 1 after each important command. This does not work well in practice.

## 106. Logging! I want to send all of my script's output to a log file. But I want to do it from inside the script. And I want to see it on the terminal too!

Normally, if you want to run a script and send its output to a logfile, you'd simply use [Redirection](https://mywiki.wooledge.org/Redirection): myscript >log 2>&1. Or to see the output on the screen _and also_ redirect to a file: myscript 2>&1 | tee log (or better still, run your script within the script(1) command if your system has it; script is intended for use with interactive shell sessions.). If you want to insert commands _into_ a script that cause it to do this kind of logging internally, without altering your invocation, then it gets trickier.

## 107. How do I add a timestamp to every line of a stream?

Adding timestamps to a stream is a challenge, because there aren't any standard tools to do it. You either have to install something specifically for it (e.g. ts from [moreutils](http://joeyh.name/code/moreutils/), or [multilog](http://cr.yp.to/daemontools/multilog.html) from daemontools), or write a filter in some programming language. Ideally, you **do not want** to fork a date(1) command for every line of input that you're logging, because it's too slow. You want to use builtins. Older versions of bash _cannot do this_. You need at least [Bash 4.2](https://mywiki.wooledge.org/BashFAQ/061) for the printf %(...)T option. Otherwise, you could write something in Perl, Python, Tcl, etc. to read lines and write them out with timestamps.

## 108. How do I wait for several spawned processes?

There are numerous ways to do this, but all of them are either limited by the available tools. I have come up with the following solutions.

## 109. How can I tell whether my script was sourced (dotted in) or executed?

There seem to be two reasons why people ask this: either they're trying to detect user errors and provide a friendly message, or they're Python programmers who want to use one of Python's most idiosyncratic features in bash.

## 110. How do I copy a file to a remote system, and specify a remote name which may contain spaces?

All of the common tools for copying files to a remote system (ssh, scp, rsync) send the filename as part of a shell command, which the remote system interprets. This makes the issue extremely complex, because the remote shell will often mangle the filename. There are at least three ways to deal with the problem: NFS, careful encoding of the filename, or submission of the filename as part of the data stream.

## 111. What is the Shellshock vulnerability in Bash?

"Shellshock" refers to two remotely-exploitable vulnerabilities in Bash, discovered in September 2014. The first vulnerability exploits the mechanism that Bash used to export and import functions, and allowed arbitrary command execution. The second vulnerability exploits a parser bug and allowed local files to be created.

## 112. What are the advantages and disadvantages of using set -u (or set -o nounset)?

Bash (like all other Bourne shell derivatives) has a feature activated by the command set -u (or set -o nounset). When this feature is in effect, any command which attempts to expand an unset variable will cause a **fatal** error (the shell immediately exits, unless it is interactive).

**Do not** attempt this with sed, awk, grep, and so on (it leads to [undesired results](https://stackoverflow.com/a/1758162)). In many cases, your best option is to write in a language that has support for XML data. If you have to use a shell script, there are a few HTML- and XML-specific tools available to parse these files for you.

## 114. How do I operate on IP addresses and netmasks?

IPv4 addresses are 32-bit unsigned integers. The "dotted quad" notation (192.168.1.2) is only one means of representing such an address. When applying netmasks, it's easier if we first convert the dotted quad format into a plain integer.

Some people like to use select because it's simple. If your own needs are extremely simple, then this may be sufficient for you. If you want your own look and feel, you can simply write a menu yourself. There is also [dialog](https://mywiki.wooledge.org/BashFAQ/040), which we won't cover on this page.

## 116. I have two files. The first one contains bad IP addresses (plus other fields). I want to remove all of these bad addresses from a second file.

This is a more generalized form of one of the questions from [FAQ 36](https://mywiki.wooledge.org/BashFAQ/036) (where the entire line is significant in each file). In this form, we're only using _part_ of each line as a key. We're going to show how to approach this kind of problem using an [associative array](https://mywiki.wooledge.org/BashFAQ/006).

## 117. I have a pipeline where a long-running command feeds into a filter. If the filter finds "foo", I want the long-running command to die.

In general this is not possible, because sibling processes (two children of the same parent) do not have any knowledge of each other. But consider the following example and answers:

## 118. How do I print the contents of an array in reverse order, or reverse an array?

First note that the concept of _order_ applies only to [indexed arrays](https://mywiki.wooledge.org/BashFAQ/005), not associative arrays. The answers would be simpler if there were no sparse arrays, but bash's arrays _can_ be sparse (non-sequential indices). So we have to introduce an extra step.

## 119. What's the difference between "cmd < file" and "cat file | cmd"? What is a UUOC?

Most of the time, these commands do the same thing, but the second one is less efficient, and it also breaks in certain rare circumstances.

## 120. How can I find out where this strange variable in my interactive shell came from?

Some variables are set by programs that run before bash, and this FAQ can't help you with those. For the variables that are set by bash reading a [dot file](https://mywiki.wooledge.org/DotFiles), if you are _not_ root, you can use bash's trace mode:

## 121. What does value too great for base mean? (Octal values in arithmetic.)

When reading numbers from files or commands and then performing arithmetic with them, leading zeroes may cause a problem:

$ echo $((09))
bash: 09: value too great for base (error token is "09")

## 122. How do I print a horizontal line of characters like ----?

There are _many_ different ways, depending on how fancy you'd like to be. This page is a restoration of content from the now-defunct bash-hackers wiki, with some additional text.

---

[CategoryShell](https://mywiki.wooledge.org/CategoryShell)