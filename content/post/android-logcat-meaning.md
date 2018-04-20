+++
title = "Android EventLogTags"
description = "android logcat 内容释义"
tags = [
    "android",
    "tech"
]
date = "2018-04-01"
topics = [
    "android",
    "tech"
]
toc = true
+++

Android logcat中event信息的含义说明。

<!-- more -->

[official doc](https://android.googlesource.com/platform/frameworks/base/+/master/services/core/java/com/android/server/am/EventLogTags.logtags)

[EventLog分析](https://blog.csdn.net/darkengine/article/details/8477502)提到的[文档](https://android.googlesource.com/platform/system/core/+/669ecc2f5e80ff924fa20ce7445354a7c5bcfd98/logcat/event.logtags)

[Android event日志打印原理](https://blog.csdn.net/yaowei514473839/article/details/53513435)

## event.logtags
```
logcat / event.logtags
android / platform / system / core / 669ecc2f5e80ff924fa20ce7445354a7c5bcfd98 / . / logcat / event.logtags
frameworks/base/services/core/java/com/android/server/am/EventLogTags.logtags

# The entries in this file map a sparse set of log tag numbers to tag names.
# This is installed on the device, in /system/etc, and parsed by logcat.
#
# Tag numbers are decimal integers, from 0 to 2^31.  (Let's leave the
# negative values alone for now.)
#
# Tag names are one or more ASCII letters and numbers or underscores, i.e.
# "[A-Z][a-z][0-9]_".  Do not include spaces or punctuation (the former
# impacts log readability, the latter makes regex searches more annoying).
#
# Tag numbers and names are separated by whitespace.  Blank lines and lines
# starting with '#' are ignored.
#
# Optionally, after the tag names can be put a description for the value(s)
# of the tag. Description are in the format
#    (<name>|data type[|data unit])
# Multiple values are separated by commas.
#
# The data type is a number from the following values:
# 1: int
# 2: long
# 3: string
# 4: list
#
# The data unit is a number taken from the following list:
# 1: Number of objects
# 2: Number of bytes
# 3: Number of milliseconds
# 4: Number of allocations
# 5: Id
# 6: Percent
# Default value for data of type int/long is 2 (bytes).
#
# TODO: generate ".java" and ".h" files with integer constants from this file.
# These are used for testing, do not modify without updating
# tests/framework-tests/src/android/util/EventLogFunctionalTest.java.
42    answer (to life the universe etc|3)
314   pi
2718  e
# "account" is the java hash of the account name
2720 sync (id|3),(event|1|5),(source|1|5),(account|1|5)
# This event is logged when the location service uploads location data.
2740 location_controller
# This event is logged when someone is deciding to force a garbage collection
2741 force_gc (reason|3)
# This event is logged on each tickle
2742 tickle (authority|3)
# contacts aggregation: time and number of contacts.
# count is negative for query phase, positive for merge phase
2747 contacts_aggregation (aggregation time|2|3), (count|1|1)
# Device boot timings.  We include monotonic clock values because the
# intrinsic event log times are wall-clock.
#
# Runtime starts:
3000 boot_progress_start (time|2|3)
# ZygoteInit class preloading starts:
3020 boot_progress_preload_start (time|2|3)
# ZygoteInit class preloading ends:
3030 boot_progress_preload_end (time|2|3)
# Dalvik VM
20003 dvm_lock_sample (process|3),(main|1|5),(thread|3),(time|1|3),(file|3),(line|1|5),(ownerfile|3),(ownerline|1|5),(sample_percent|1|6)
75000 sqlite_mem_alarm_current (current|1|2)
75001 sqlite_mem_alarm_max (max|1|2)
75002 sqlite_mem_alarm_alloc_attempt (attempts|1|4)
75003 sqlite_mem_released (Memory released|1|2)
75004 sqlite_db_corrupt (Database file corrupt|3)
50000 menu_item_selected (Menu type where 0 is options and 1 is context|1|5),(Menu item title|3)
50001 menu_opened (Menu type where 0 is options and 1 is context|1|5)
# HSM wifi state change
# Hierarchical state class name (as defined in WifiStateTracker.java)
# Logged on every state change in the hierarchical state machine
50021 wifi_state_changed (wifi_state|3)
# HSM wifi event
# [31-16] Reserved for future use
# [15 - 0] HSM event (as defined in WifiStateTracker.java)
# Logged when an event is handled in a hierarchical state
50022 wifi_event_handled (wifi_event|1|5)
# Supplicant state change
# [31-13] Reserved for future use
# [8 - 0] Supplicant state (as defined in SupplicantState.java)
# Logged when the supplicant switches to a new state
50023 wifi_supplicant_state_changed (supplicant_state|1|5)
# Database operation samples.
# db: the filename of the database
# sql: the executed query (without query args)
# time: cpu time millis (not wall time), including lock acquisition
# blocking_package: if this is on a main thread, the package name, otherwise ""
# sample_percent: the percent likelihood this query was logged
52000 db_sample (db|3),(sql|3),(time|1|3),(blocking_package|3),(sample_percent|1|6)
# http request/response stats
52001 http_stats (useragent|3),(response|2|3),(processing|2|3),(tx|1|2),(rx|1|2)
60000 viewroot_draw (Draw time|1|3)
60001 viewroot_layout (Layout time|1|3)
60002 view_build_drawing_cache (View created drawing cache|1|5)
60003 view_use_drawing_cache (View drawn using bitmap cache|1|5)
# graphics timestamp
# 60100 - 60199 reserved for surfaceflinger
# 0 for screen off, 1 for screen on, 2 for key-guard done
70000 screen_toggled (screen_state|1|5)
# aggregation service
70200 aggregation (aggregation time|2|3)
70201 aggregation_test (field1|1|2),(field2|1|2),(field3|1|2),(field4|1|2),(field5|1|2)
# libc failure logging
80100 bionic_event_memcpy_buffer_overflow (uid|1)
80105 bionic_event_strcat_buffer_overflow (uid|1)
80110 bionic_event_memmov_buffer_overflow (uid|1)
80115 bionic_event_strncat_buffer_overflow (uid|1)
80120 bionic_event_strncpy_buffer_overflow (uid|1)
80125 bionic_event_memset_buffer_overflow (uid|1)
80130 bionic_event_strcpy_buffer_overflow (uid|1)
80200 bionic_event_strcat_integer_overflow (uid|1)
80205 bionic_event_strncat_integer_overflow (uid|1)
80300 bionic_event_resolver_old_response (uid|1)
80305 bionic_event_resolver_wrong_server (uid|1)
80310 bionic_event_resolver_wrong_query (uid|1)
# libcore failure logging
90100 exp_det_cert_pin_failure (certs|4)
1397638484 snet_event_log (subtag|3) (uid|1) (message|3)
# NOTE - the range 1000000-2000000 is reserved for partners and others who
# want to define their own log tags without conflicting with the core platform.
```

## EventLogTags.logtags

```
https://android.googlesource.com/platform/frameworks/base/+/master/services/core/java/com/android/server/am/EventLogTags.logtags

frameworks/base/services/core/java/com/android/server/am/EventLogTags.logtags

# See system/core/logcat/event.logtags for a description of the format of this file.
option java_package com.android.server.am
2719 configuration_changed (config mask|1|5)
2721 cpu (total|1|6),(user|1|6),(system|1|6),(iowait|1|6),(irq|1|6),(softirq|1|6)
# ActivityManagerService.systemReady() starts:
3040 boot_progress_ams_ready (time|2|3)
# ActivityManagerService calls enableScreenAfterBoot():
3050 boot_progress_enable_screen (time|2|3)
# Do not change these names without updating the checkin_events setting in
# google3/googledata/wireless/android/provisioning/gservices.config !!
#
# An activity is being finished:
30001 am_finish_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3),(Reason|3)
# A task is being brought to the front of the screen:
30002 am_task_to_front (User|1|5),(Task|1|5)
# An existing activity is being given a new intent:
30003 am_new_intent (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3),(Action|3),(MIME Type|3),(URI|3),(Flags|1|5)
# A new task is being created:
30004 am_create_task (User|1|5),(Task ID|1|5)
# A new activity is being created in an existing task:
30005 am_create_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3),(Action|3),(MIME Type|3),(URI|3),(Flags|1|5)
# An activity has been resumed into the foreground but was not already running:
30006 am_restart_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3)
# An activity has been resumed and is now in the foreground:
30007 am_resume_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3)
# Application Not Responding
30008 am_anr (User|1|5),(pid|1|5),(Package Name|3),(Flags|1|5),(reason|3)
# Activity launch time
30009 am_activity_launch_time (User|1|5),(Token|1|5),(Component Name|3),(time|2|3)
# Application process bound to work
30010 am_proc_bound (User|1|5),(PID|1|5),(Process Name|3)
# Application process died
30011 am_proc_died (User|1|5),(PID|1|5),(Process Name|3),(OomAdj|1|5),(ProcState|1|5)
# The Activity Manager failed to pause the given activity.
30012 am_failed_to_pause (User|1|5),(Token|1|5),(Wanting to pause|3),(Currently pausing|3)
# Attempting to pause the current activity
30013 am_pause_activity (User|1|5),(Token|1|5),(Component Name|3)
# Application process has been started
30014 am_proc_start (User|1|5),(PID|1|5),(UID|1|5),(Process Name|3),(Type|3),(Component|3)
# An application process has been marked as bad
30015 am_proc_bad (User|1|5),(UID|1|5),(Process Name|3)
# An application process that was bad is now marked as good
30016 am_proc_good (User|1|5),(UID|1|5),(Process Name|3)
# Reporting to applications that memory is low
30017 am_low_memory (Num Processes|1|1)
# An activity is being destroyed:
30018 am_destroy_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3),(Reason|3)
# An activity has been relaunched, resumed, and is now in the foreground:
30019 am_relaunch_resume_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3)
# An activity has been relaunched:
30020 am_relaunch_activity (User|1|5),(Token|1|5),(Task ID|1|5),(Component Name|3)
# The activity's onPause has been called.
30021 am_on_paused_called (User|1|5),(Component Name|3),(Reason|3)
# The activity's onResume has been called.
30022 am_on_resume_called (User|1|5),(Component Name|3),(Reason|3)
# Kill a process to reclaim memory.
30023 am_kill (User|1|5),(PID|1|5),(Process Name|3),(OomAdj|1|5),(Reason|3)
# Discard an undelivered serialized broadcast (timeout/ANR/crash)
30024 am_broadcast_discard_filter (User|1|5),(Broadcast|1|5),(Action|3),(Receiver Number|1|1),(BroadcastFilter|1|5)
30025 am_broadcast_discard_app (User|1|5),(Broadcast|1|5),(Action|3),(Receiver Number|1|1),(App|3)
# A service is being created
30030 am_create_service (User|1|5),(Service Record|1|5),(Name|3),(UID|1|5),(PID|1|5)
# A service is being destroyed
30031 am_destroy_service (User|1|5),(Service Record|1|5),(PID|1|5)
# A process has crashed too many times, it is being cleared
30032 am_process_crashed_too_much (User|1|5),(Name|3),(PID|1|5)
# An unknown process is trying to attach to the activity manager
30033 am_drop_process (PID|1|5)
# A service has crashed too many times, it is being stopped
30034 am_service_crashed_too_much (User|1|5),(Crash Count|1|1),(Component Name|3),(PID|1|5)
# A service is going to be restarted after its process went away
30035 am_schedule_service_restart (User|1|5),(Component Name|3),(Time|2|3)
# A client was waiting for a content provider, but its process was lost
30036 am_provider_lost_process (User|1|5),(Package Name|3),(UID|1|5),(Name|3)
# The activity manager gave up on a new process taking too long to start
30037 am_process_start_timeout (User|1|5),(PID|1|5),(UID|1|5),(Process Name|3)
# Unhandled exception
30039 am_crash (User|1|5),(PID|1|5),(Process Name|3),(Flags|1|5),(Exception|3),(Message|3),(File|3),(Line|1|5)
# Log.wtf() called
30040 am_wtf (User|1|5),(PID|1|5),(Process Name|3),(Flags|1|5),(Tag|3),(Message|3)
# User switched
30041 am_switch_user (id|1|5)
# Activity fully drawn time
30042 am_activity_fully_drawn_time (User|1|5),(Token|1|5),(Component Name|3),(time|2|3)
# Activity set to resumed
30043 am_set_resumed_activity (User|1|5),(Component Name|3),(Reason|3)
# Stack focus
30044 am_focused_stack (User|1|5),(Focused Stack Id|1|5),(Last Focused Stack Id|1|5),(Reason|3)
# Running pre boot receiver
30045 am_pre_boot (User|1|5),(Package|3)
# Report collection of global memory state
30046 am_meminfo (Cached|2|2),(Free|2|2),(Zram|2|2),(Kernel|2|2),(Native|2|2)
# Report collection of memory used by a process
30047 am_pss (Pid|1|5),(UID|1|5),(Process Name|3),(Pss|2|2),(Uss|2|2),(SwapPss|2|2)
# Attempting to stop an activity
30048 am_stop_activity (User|1|5),(Token|1|5),(Component Name|3)
# The activity's onStop has been called.
30049 am_on_stop_called (User|1|5),(Component Name|3),(Reason|3)
# Report changing memory conditions (Values are ProcessStats.ADJ_MEM_FACTOR* constants)
30050 am_mem_factor (Current|1|5),(Previous|1|5)
# UserState has changed
30051 am_user_state_changed (id|1|5),(state|1|5)
# Note when any processes of a uid have started running
30052 am_uid_running (UID|1|5)
# Note when all processes of a uid have stopped.
30053 am_uid_stopped (UID|1|5)
# Note when the state of a uid has become active.
30054 am_uid_active (UID|1|5)
# Note when the state of a uid has become idle (background check enforced).
30055 am_uid_idle (UID|1|5)
# Note when a service is being forcibly stopped because its app went idle.
30056 am_stop_idle_service (UID|1|5),(Component Name|3)
```