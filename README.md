# WordPress Cron Troubleshooting

A practical guide for diagnosing and fixing WordPress WP-Cron problems, including missed scheduled posts, failed cron events, WooCommerce scheduled actions, stuck background tasks, server cron configuration, plugin conflicts, and WordPress background processing.

---

## Overview

WordPress uses scheduled tasks for many background operations.

Common examples include:

* Publishing scheduled posts
* Checking for updates
* Sending scheduled notifications
* Processing background tasks
* Running plugin maintenance jobs
* Processing WooCommerce scheduled actions
* Cleaning temporary data
* Running recurring application tasks

A cron-related problem can affect a website even when the frontend appears to work normally.

Typical symptoms include:

```text
Scheduled posts are not published
        ↓
Cron events are not running
        ↓
Plugin background tasks remain pending
        ↓
WooCommerce scheduled actions accumulate
        ↓
Site functionality becomes delayed
```

The purpose of this repository is to provide a structured troubleshooting workflow for identifying the cause of WordPress cron failures.

---

## What Is WP-Cron?

WP-Cron is WordPress's scheduling system.

It allows WordPress and plugins to register scheduled events that should run at a particular time or on a recurring schedule.

Examples include:

```text
Hourly tasks
Daily tasks
Recurring maintenance
Scheduled publishing
Plugin updates
Background processing
WooCommerce tasks
```

WP-Cron is different from a traditional operating-system cron service.

Traditional cron runs according to the server's scheduler.

WP-Cron normally depends on WordPress requests to initiate scheduled processing.

---

## WP-Cron vs Server Cron

### WP-Cron

```text
Website request
      ↓
WordPress
      ↓
WP-Cron
      ↓
Scheduled event
```

### Server Cron

```text
Operating system
      ↓
Cron daemon
      ↓
wp-cron.php / WP-CLI
      ↓
WordPress
      ↓
Scheduled events
```

For sites with significant traffic or important scheduled workloads, a real server cron can provide more predictable execution than relying entirely on visitor requests.

---

## Common WP-Cron Problems

| Symptom                                   | Possible Cause                    |
| ----------------------------------------- | --------------------------------- |
| Scheduled posts remain pending            | WP-Cron not running               |
| Cron events are overdue                   | Cron spawning failure             |
| WooCommerce tasks remain pending          | Action Scheduler issue            |
| Tasks repeatedly fail                     | Plugin/PHP/database error         |
| Cron runs very slowly                     | Server or application performance |
| Cron works manually but not automatically | Scheduler configuration           |
| WP-Cron test fails                        | Loopback or HTTP problem          |
| Cron events disappear                     | Plugin/theme behavior             |
| Large number of overdue events            | Cron backlog                      |
| CPU usage increases during cron           | Heavy scheduled tasks             |
| Background jobs never finish              | Failed or blocked workers         |
| Cron works after page visits              | Low traffic or disabled cron      |

---

# 1. Identify the Exact Failure

Before changing configuration, determine which type of cron problem is occurring.

Ask:

1. Which event is failing?
2. What is the hook name?
3. When was it supposed to run?
4. Is it overdue?
5. Does it run manually?
6. Does WP-Cron spawn successfully?
7. Is `DISABLE_WP_CRON` enabled?
8. Is a real server cron configured?
9. Are there PHP errors?
10. Are there plugin or theme conflicts?
11. Is the database healthy?
12. Are scheduled tasks accumulating?

Start with observation instead of immediately deleting cron events.

---

# 2. Check WordPress Cron Events

If WP-CLI is available, list scheduled events:

```bash
wp cron event list
```

For a more compact view:

```bash
wp cron event list --fields=hook,next_run
```

JSON output can also be useful:

```bash
wp cron event list --fields=hook,next_run --format=json
```

This helps identify:

* Hook names
* Scheduled execution times
* Overdue events
* Recurring events
* Large numbers of scheduled jobs

WP-CLI officially supports listing, scheduling, running, deleting, and unscheduling cron events.

---

# 3. Check Available Cron Schedules

List registered schedules:

```bash
wp cron schedule list
```

Example:

```text
+------------+-------------+----------+
| name       | display     | interval |
+------------+-------------+----------+
| hourly     | Once Hourly  | 3600     |
| twicedaily | Twice Daily  | 43200    |
| daily      | Once Daily   | 86400    |
+------------+-------------+----------+
```

Custom plugins can register additional schedules.

Look for unexpected or missing schedules.

Official WP-CLI documentation provides the `wp cron schedule list` command for this purpose.

---

# 4. Test WP-Cron

Run:

```bash
wp cron test
```

A successful result should indicate that WP-Cron spawning is working.

The command checks whether WP-Cron is disabled, checks the alternate cron setting, and attempts to spawn WP-Cron over HTTP.

If the test fails, investigate the underlying reason instead of simply rerunning it repeatedly.

---

# 5. Check `DISABLE_WP_CRON`

Inspect `wp-config.php`.

Look for:

```php
define( 'DISABLE_WP_CRON', true );
```

If this is enabled, normal WP-Cron spawning is disabled.

This can be intentional when a server-level cron job is configured.

For example:

```text
DISABLE_WP_CRON = true
        ↓
Server cron must execute WordPress cron
```

Do not remove the constant blindly.

First determine whether a replacement cron mechanism already exists.

---

# 6. Check `ALTERNATE_WP_CRON`

Look for:

```php
define( 'ALTERNATE_WP_CRON', true );
```

This changes how WordPress attempts to trigger cron processing.

If this setting exists unexpectedly, investigate why it was enabled.

Do not change production cron configuration without understanding the current environment.

---

# 7. Check `wp-cron.php`

WordPress includes:

```text
wp-cron.php
```

Normally located in the WordPress root directory.

Example:

```text
public_html/
├── wp-admin/
├── wp-content/
├── wp-includes/
├── index.php
├── wp-config.php
└── wp-cron.php
```

Verify that the file exists and has not been removed or modified unexpectedly.

---

# 8. Check WordPress Loopback Requests

WP-Cron can depend on HTTP communication with the site itself.

Problems with loopback requests can prevent scheduled tasks from spawning.

Potential causes include:

* Firewall rules
* Security plugins
* Authentication requirements
* DNS problems
* SSL problems
* Reverse proxy configuration
* Cloudflare configuration
* Server networking
* PHP errors
* Incorrect site URL
* Hosting restrictions

Test the site's HTTP response:

```bash
curl -I https://example.com/
```

For a direct cron endpoint test:

```bash
curl -I https://example.com/wp-cron.php
```

Do not assume that a successful HTTP response alone proves that cron processing is healthy.

---

# 9. Check Site URL Configuration

Verify WordPress URL settings:

```bash
wp option get home
wp option get siteurl
```

Unexpected differences can cause loopback or cron problems.

Common issues include:

```text
http://example.com
https://example.com
www.example.com
example.com
```

Make sure the configured URLs match the intended production environment.

---

# 10. Check HTTPS and SSL

Cron spawning can be affected by SSL problems.

Check:

```bash
curl -I https://example.com/
```

Look for:

* Certificate errors
* Redirect loops
* Incorrect hostname
* HTTP-to-HTTPS problems
* Proxy-related behavior

A browser may appear to work while server-side HTTP requests fail.

---

# 11. Check Redirects

Inspect redirects:

```bash
curl -IL https://example.com/wp-cron.php
```

Look for:

```text
HTTP/1.1 301
HTTP/1.1 302
HTTP/1.1 200
```

A complicated redirect chain can interfere with internal requests.

Check:

* HTTP to HTTPS
* www to non-www
* Non-www to www
* Cloudflare redirects
* WordPress redirects
* Security plugin redirects

---

# 12. Check Server Firewall

A firewall may block or interfere with internal or outbound HTTP requests.

Check:

```text
Firewall
CSF
ModSecurity
Web Application Firewall
Cloudflare
Hosting security rules
```

On managed hosting, ask the provider whether loopback or outbound HTTP requests are restricted.

Avoid disabling security controls globally just to make cron work.

---

# 13. Check PHP Errors

Cron tasks execute PHP code.

A plugin or theme fatal error can stop a scheduled task.

Check:

```text
PHP error logs
WordPress debug.log
PHP-FPM logs
Web server logs
Plugin logs
WooCommerce logs
```

Common problems include:

```text
Fatal error
Memory exhausted
Maximum execution time exceeded
Undefined function
Database error
API timeout
HTTP request failure
```

---

# 14. Enable WordPress Debug Logging Temporarily

When appropriate:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

The default debug log location is:

```text
wp-content/debug.log
```

After troubleshooting, review whether debugging should remain enabled.

Do not display PHP errors to visitors on a production website.

---

# 15. Check WP-CLI Availability

Run:

```bash
wp --info
```

Then:

```bash
wp cli version
```

Test WordPress:

```bash
wp core version
```

If WP-CLI is unavailable, use the WordPress dashboard and server tools instead.

---

# 16. Run Due Cron Events Manually

To execute events that are currently due:

```bash
wp cron event run --due-now
```

This is useful for determining whether the underlying scheduled tasks can execute successfully.

WP-CLI supports `--due-now` for running all cron hooks that are currently due.

If the command produces an error, investigate that error before changing cron configuration.

---

# 17. Run a Specific Cron Hook

First identify the hook:

```bash
wp cron event list
```

Then run the specific hook:

```bash
wp cron event run HOOK_NAME
```

Example:

```bash
wp cron event run my_custom_hook
```

Do not run unknown production hooks repeatedly.

Some hooks can perform expensive or destructive operations depending on the plugin that registered them.

---

# 18. Check Overdue Events

List scheduled events:

```bash
wp cron event list
```

Look for events whose `next_run` time is significantly in the past.

Example:

```text
Hook                  Next Run
--------------------------------
wp_version_check      2 hours ago
custom_sync_job       3 hours ago
woocommerce_task      4 hours ago
```

A large backlog usually indicates that cron execution is not keeping up.

---

# 19. Check Cron Frequency

WordPress schedules can include:

```text
hourly
twicedaily
daily
```

Plugins can register additional intervals.

Check:

```bash
wp cron schedule list
```

Do not create extremely frequent schedules without considering server load.

For example, scheduling expensive operations every minute can create unnecessary database and CPU activity.

---

# 20. Server Cron Configuration

A common production configuration is:

```text
DISABLE_WP_CRON = true
        ↓
Server cron
        ↓
wp-cron.php
```

The exact command depends on the hosting environment.

A generic example is:

```bash
*/5 * * * * php /path/to/wordpress/wp-cron.php
```

Another approach can use WP-CLI:

```bash
*/5 * * * * cd /path/to/wordpress && wp cron event run --due-now
```

Use the method appropriate for the site's environment.

Do not add both mechanisms blindly.

---

# 21. Avoid Duplicate Cron Systems

A common configuration mistake is having:

```text
WP-Cron
+
Server Cron
+
Additional Plugin Cron
```

all processing the same workloads without a clear design.

This can cause:

* Duplicate processing
* Increased server load
* Race conditions
* Duplicate API requests
* Repeated tasks

Before changing cron configuration, document what is currently executing scheduled events.

---

# 22. Check Cron Locking

WordPress uses mechanisms to prevent overlapping cron processing.

If a cron process becomes stuck, subsequent processing may be delayed.

Potential causes include:

* Long-running PHP tasks
* External API timeouts
* Database locks
* Slow queries
* Large imports
* Backup processes
* Plugin bugs

Do not manually delete internal cron locks unless you understand the current process and have confirmed that no legitimate cron worker is running.

---

# 23. WooCommerce Scheduled Actions

WooCommerce uses Action Scheduler for background processing.

Examples include:

```text
Order-related tasks
Subscription processing
Payment-related tasks
Webhooks
Emails
Data cleanup
Plugin integrations
Recurring jobs
```

Check the WooCommerce scheduled actions interface when available.

Typical location:

```text
WooCommerce
→ Status
→ Scheduled Actions
```

Depending on the WooCommerce version and installed extensions, the interface may vary.

---

# 24. Action Scheduler Status

Common statuses include:

```text
Pending
Complete
Failed
Canceled
```

A large number of pending actions may indicate:

* Cron is not running
* Workers are overloaded
* PHP errors
* Database problems
* External API failures
* Plugin conflicts

A large number of failed actions requires investigation rather than simply deleting the failed records.

---

# 25. WooCommerce Cron Problems

If WooCommerce tasks are not processing:

Check:

1. WP-Cron status
2. Scheduled Actions
3. Failed actions
4. PHP errors
5. Database health
6. Payment gateway logs
7. Webhook logs
8. Plugin conflicts
9. Server resources
10. External API connectivity

Do not assume every WooCommerce background task problem is a WP-Cron problem.

---

# 26. Check Database Performance

Cron tasks frequently interact with the database.

Look for:

```text
Slow queries
Database errors
Large tables
Table locks
High database CPU
Connection failures
Database timeouts
```

For WordPress sites with heavy WooCommerce usage, scheduled tasks can generate substantial database activity.

---

# 27. Check Server Resources

Monitor:

```text
CPU
RAM
Disk I/O
Disk space
PHP workers
Database connections
Load average
Network activity
```

A cron task may technically be running but taking too long because the server is overloaded.

---

# 28. Check PHP-FPM

On servers using PHP-FPM, check:

```text
pm.max_children
pm.max_requests
request_terminate_timeout
memory_limit
max_execution_time
```

A busy PHP-FPM pool can delay cron requests.

Do not increase worker limits blindly.

Higher limits can increase memory consumption and make an already overloaded server less stable.

---

# 29. Check PHP Memory

Cron tasks may require more memory than normal frontend requests.

Common errors include:

```text
Allowed memory size exhausted
Fatal error: Out of memory
```

Check the active PHP configuration:

```bash
php -i | grep memory_limit
```

For hosting environments with multiple PHP versions, verify that the command refers to the same PHP version used by WordPress.

---

# 30. Check Execution Time

Long-running tasks can exceed:

```text
max_execution_time
```

Other limits may also matter:

```text
request_terminate_timeout
proxy timeout
web server timeout
PHP-FPM timeout
external API timeout
```

Do not increase every timeout automatically.

First identify which process is actually timing out.

---

# 31. Check Plugin Conflicts

Plugins frequently register cron events.

A plugin can also:

* Register invalid schedules
* Create excessive events
* Fail during execution
* Trigger expensive database queries
* Call unavailable APIs
* Generate fatal PHP errors

If cron problems started after a plugin update:

1. Record the current configuration.
2. Back up the site.
3. Test the affected event.
4. Review plugin logs.
5. Test with the suspected plugin disabled where safe.
6. Compare results.

Avoid disabling critical production plugins without understanding their function.

---

# 32. Check Theme Code

Themes can also register scheduled events.

Search custom code for:

```php
wp_schedule_event()
wp_schedule_single_event()
wp_next_scheduled()
wp_clear_scheduled_hook()
```

A badly implemented scheduling function can create duplicate events.

For example, scheduling an event on every request without checking whether it already exists can create unnecessary cron entries.

---

# 33. Correct Scheduling Pattern

A common pattern is:

```php
if ( ! wp_next_scheduled( 'my_custom_hook' ) ) {
    wp_schedule_event( time(), 'hourly', 'my_custom_hook' );
}
```

Then register the callback:

```php
add_action( 'my_custom_hook', 'my_custom_function' );

function my_custom_function() {
    // Scheduled task.
}
```

The exact implementation should be adapted to the plugin or application.

Avoid directly modifying WordPress core files.

---

# 34. Single Scheduled Events

For one-time tasks, WordPress supports single scheduled events.

Conceptually:

```php
wp_schedule_single_event(
    time() + HOUR_IN_SECONDS,
    'my_custom_hook'
);
```

This can be useful for delayed application tasks.

Always ensure that duplicate scheduling is prevented when appropriate.

---

# 35. Inspect Cron Events Before Deleting Them

Before deleting an event:

```bash
wp cron event list
```

Identify:

```text
Hook
Arguments
Next run
Recurrence
Plugin responsible
```

Only remove events that you understand.

Deleting a legitimate plugin event can break functionality.

---

# 36. Delete a Specific Cron Event

WP-CLI provides:

```bash
wp cron event delete HOOK_NAME
```

This can remove scheduled events for the specified hook.

Use this carefully.

Do not use broad deletion commands as a first troubleshooting step.

---

# 37. Unschedule an Event

WP-CLI also provides:

```bash
wp cron event unschedule HOOK_NAME
```

Use this only when you understand which scheduled event is being removed and why.

If a plugin owns the event, disabling or correcting the plugin may be a better long-term solution.

---

# 38. Schedule Testing Events

For controlled testing, WP-CLI can schedule an event:

```bash
wp cron event schedule cron_test
```

A recurring test can be scheduled with:

```bash
wp cron event schedule cron_test now hourly
```

WP-CLI documents both one-time and recurring event scheduling.

Do not leave unnecessary test events on production sites.

---

# 39. Low-Traffic Websites

WP-Cron can be problematic on sites with very little traffic because scheduled processing may depend on incoming requests.

For example:

```text
Scheduled event
       ↓
No visitors
       ↓
No cron spawning opportunity
       ↓
Event remains overdue
```

A server-level cron can provide a more predictable execution mechanism.

---

# 40. High-Traffic Websites

High-traffic sites can have the opposite problem.

Frequent requests may repeatedly trigger cron spawning.

Potential symptoms include:

* Increased PHP requests
* Increased CPU usage
* More concurrent background processing
* Database load
* Slower response times

For high-traffic websites, carefully designed server-level cron execution may reduce unnecessary cron spawning.

---

# 41. WP-Cron and Caching

Caching normally should not prevent WordPress cron execution directly, but reverse proxies, security layers, or unusual server configurations can interfere with internal requests.

Check:

```text
Cloudflare
Reverse proxy
Page cache
Object cache
Security plugin
WAF
Server firewall
```

Cron requests should be treated according to the site's architecture.

---

# 42. WP-Cron and Cloudflare

If a site uses Cloudflare, check:

* DNS configuration
* SSL mode
* WAF rules
* Bot protection
* Rate limiting
* Redirect rules
* Access rules

A security rule that blocks server-to-server requests can affect cron spawning.

Do not disable Cloudflare security globally.

Identify the specific rule responsible first.

---

# 43. Check External API Calls

Many scheduled tasks communicate with external services.

Examples:

```text
Payment gateways
Shipping APIs
Email services
Marketing APIs
Analytics services
Inventory APIs
Webhooks
```

If the API is slow or unavailable, the cron task may fail or take too long.

Check:

```text
HTTP status
Response time
API authentication
API rate limits
Timeouts
DNS resolution
SSL verification
```

---

# 44. Check Webhooks

Some background processing depends on webhooks.

If a webhook is failing:

```text
External service
       ↓
Webhook
       ↓
WordPress
       ↓
Plugin
       ↓
Scheduled task
```

Check the webhook logs before changing cron configuration.

---

# 45. Cron and Backups

Backup plugins may schedule:

```text
Database backups
File backups
Remote uploads
Cleanup tasks
Retention jobs
```

Large backup jobs can consume significant:

```text
CPU
RAM
Disk I/O
Network bandwidth
```

If cron problems occur during backup windows, compare the timing.

Do not disable backups simply to improve cron performance.

---

# 46. Cron and Security Plugins

Security plugins may schedule:

```text
Scans
Log cleanup
Database cleanup
Security reports
IP list updates
```

A security plugin can therefore create a significant number of scheduled tasks.

Check the plugin's documentation and logs before modifying its cron events.

---

# 47. Cron and Object Cache

Object caching can affect application behavior when plugins depend on cached state.

Check:

```text
Redis
Memcached
Persistent object cache
Cache plugins
```

If a cron task behaves differently from a normal web request, compare the cache configuration.

Do not flush production caches repeatedly without understanding the impact.

---

# 48. Multisite Cron

For WordPress multisite installations, cron behavior may need to be checked at the network and individual-site levels.

WP-CLI supports running due cron events across a multisite network:

```bash
wp cron event run --due-now --network
```

Use this only when working on a multisite environment and you understand the scope of the command.

---

# 49. Safe Troubleshooting Workflow

Use the following sequence:

```text
1. Reproduce the problem
        ↓
2. Identify affected cron hook
        ↓
3. List scheduled events
        ↓
4. Run wp cron test
        ↓
5. Check DISABLE_WP_CRON
        ↓
6. Check loopback/HTTP requests
        ↓
7. Check PHP errors
        ↓
8. Check WooCommerce scheduled actions
        ↓
9. Check server resources
        ↓
10. Check database performance
        ↓
11. Check plugin/theme conflicts
        ↓
12. Test the affected event manually
        ↓
13. Apply the smallest safe fix
        ↓
14. Retest
        ↓
15. Monitor scheduled tasks
```

---

# 50. Troubleshooting Checklist

## WordPress

* [ ] WordPress loads normally
* [ ] `wp-cron.php` exists
* [ ] WP-Cron configuration checked
* [ ] `DISABLE_WP_CRON` checked
* [ ] `ALTERNATE_WP_CRON` checked
* [ ] Site URL checked
* [ ] WP-CLI available
* [ ] Cron events listed
* [ ] Cron test completed

## HTTP and Networking

* [ ] Site responds over HTTPS
* [ ] Loopback request works
* [ ] No redirect loop
* [ ] Firewall checked
* [ ] WAF checked
* [ ] Cloudflare rules checked
* [ ] DNS resolution works

## PHP

* [ ] PHP errors checked
* [ ] Memory limit checked
* [ ] Execution time checked
* [ ] PHP-FPM checked
* [ ] Correct PHP version verified

## WooCommerce

* [ ] Scheduled Actions checked
* [ ] Pending actions reviewed
* [ ] Failed actions reviewed
* [ ] Payment logs checked
* [ ] Webhook logs checked
* [ ] Plugin conflicts checked

## Server

* [ ] CPU checked
* [ ] RAM checked
* [ ] Disk I/O checked
* [ ] Disk space checked
* [ ] Database performance checked
* [ ] Server cron checked
* [ ] Duplicate cron systems ruled out

---

# 51. Quick Diagnostic Table

| Symptom                                   | First Things to Check                  |
| ----------------------------------------- | -------------------------------------- |
| Scheduled post not published              | WP-Cron events and cron test           |
| Cron events overdue                       | WP-Cron spawning and server cron       |
| `wp cron test` fails                      | Loopback, HTTPS, firewall              |
| Cron works manually but not automatically | Scheduler configuration                |
| WooCommerce tasks pending                 | Scheduled Actions and cron             |
| WooCommerce tasks failing                 | Failed actions and PHP logs            |
| Cron creates high CPU                     | Number and frequency of events         |
| Cron causes high PHP usage                | PHP-FPM and task duration              |
| Cron stops after plugin update            | Plugin conflict                        |
| Cron stops after PHP update               | PHP compatibility/errors               |
| Cron works after site visit               | Low traffic or scheduler configuration |
| Server cron runs but tasks fail           | WordPress/PHP/application errors       |
| Large cron backlog                        | Worker capacity or failed tasks        |
| Repeated cron events                      | Duplicate scheduling                   |
| External task fails                       | API/network/authentication             |

---

# 52. Document the Investigation

For professional troubleshooting, record:

```text
Website:
Date:
WordPress Version:
WooCommerce Version:
PHP Version:
Hosting Environment:
WP-Cron Enabled:
DISABLE_WP_CRON:
Server Cron:
Affected Hook:
Next Run:
Observed Error:
PHP Error:
Database Error:
Plugin Involved:
Changes Made:
Test Result:
Final Resolution:
```

Documentation makes future troubleshooting significantly easier.

---

# 53. Prevention Best Practices

For reliable scheduled task processing:

* Keep WordPress updated.
* Keep plugins and themes updated.
* Monitor scheduled events.
* Avoid unnecessary high-frequency cron jobs.
* Prevent duplicate event registration.
* Monitor WooCommerce Scheduled Actions.
* Monitor PHP errors.
* Monitor database performance.
* Keep server resources within safe limits.
* Use server cron where appropriate.
* Document cron architecture.
* Test cron after major hosting changes.
* Monitor external API dependencies.
* Keep regular backups.
* Avoid deleting cron events without investigation.

---

# 54. Production Safety Guidelines

Before changing cron configuration on a production website:

1. Create a backup.
2. Record the current configuration.
3. Identify the affected hook.
4. Determine which plugin or component owns it.
5. Check logs.
6. Make one controlled change.
7. Test the result.
8. Monitor the next scheduled execution.

Avoid:

```text
Deleting all cron events
Disabling all plugins
Changing PHP versions without testing
Disabling security controls globally
Deleting database records blindly
Increasing every server limit
```

---

# 55. Official References

### WordPress Developer Resources

* WP-Cron Function Reference:
  [https://developer.wordpress.org/reference/functions/wp_cron/](https://developer.wordpress.org/reference/functions/wp_cron/)

* WP-CLI Cron Commands:
  [https://developer.wordpress.org/cli/commands/cron/](https://developer.wordpress.org/cli/commands/cron/)

* WP-CLI Cron Event Commands:
  [https://developer.wordpress.org/cli/commands/cron/event/](https://developer.wordpress.org/cli/commands/cron/event/)

* WP-CLI Cron Test:
  [https://developer.wordpress.org/cli/commands/cron/test/](https://developer.wordpress.org/cli/commands/cron/test/)

* WP-CLI Cron Schedule:
  [https://developer.wordpress.org/cli/commands/cron/schedule/list/](https://developer.wordpress.org/cli/commands/cron/schedule/list/)

---

# 56. Final Diagnostic Flow

Use this flow when investigating a production cron problem:

```text
                       CRON PROBLEM
                            |
                            v
                 Is the event scheduled?
                       /          \
                     NO            YES
                     |              |
                     v              v
              Check plugin      Is it overdue?
              configuration       /       \
                                 NO        YES
                                 |          |
                                 v          v
                            Monitor it   Test WP-Cron
                                            |
                                            v
                                  Does wp cron test pass?
                                       /          \
                                     NO            YES
                                     |              |
                                     v              v
                              Check loopback     Run event
                              HTTPS/firewall      manually
                                                     |
                                                     v
                                            Does the event fail?
                                             /             \
                                           YES             NO
                                            |               |
                                            v               v
                                      Check PHP,        Check server
                                      plugin, DB        scheduler
                                      and API           configuration
                                            |
                                            v
                                      Apply safe fix
                                            |
                                            v
                                          Retest
                                            |
                                            v
                                         Monitor
```

---

## Important Notes

This repository is intended as a practical troubleshooting reference for WordPress administrators, developers, hosting providers, and support teams.

Cron problems can originate from WordPress, plugins, themes, PHP, databases, web servers, firewalls, external APIs, or server scheduling systems.

Do not assume that every scheduled-task problem is caused by WP-Cron itself.

Always identify the failing component before making production changes.

For production websites:

1. Take a backup before major changes.
2. Record the existing configuration.
3. Make one controlled change at a time.
4. Test after each change.
5. Monitor subsequent scheduled executions.
6. Document the final resolution.

---

## Contributing

Contributions are welcome.

If you find an additional WordPress cron troubleshooting scenario:

1. Open an issue.
2. Explain the symptoms.
3. Include the relevant error message.
4. Describe the environment.
5. Remove sensitive information.
6. Explain the confirmed solution when available.

Do not publish passwords, API keys, private logs, customer information, or other sensitive server data.

---

## License

This documentation is provided for educational and troubleshooting purposes.

Use the procedures carefully and adapt them to your own WordPress installation, hosting environment, PHP configuration, server scheduler, and application architecture.

[1]: https://developer.wordpress.org/cli/commands/cron/?utm_source=chatgpt.com "wp cron – WP-CLI Command | Developer.WordPress.org"
