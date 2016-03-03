# DICrontab
This is a php class implement a timetable service using format string like linux crontab, dependence on swoole-extension.

Thanks to the project [swoole-crontab](https://github.com/osgochina/swoole-crontab), I learned how to analyze the crontab string.

# Quick start
```
<?php
include './CrontabTicker.php'

$crontab = new \CrontabTicker();
$crontab->Set('* */20 * * *')
	->Called(function ($userParams)
	{
		echo 'crontab called';
		return false;//return false int callback function if you want to cancle this cron.
        },
        $userParams);
//The callback function will be called every 20 minitues.
```

# Description
While working my on swoole-framework [DIServer-framwork](https://github.com/szyhf/DIServer), I find is not easy to implement some timer job like send a message to every clients on every Monday to Friday, swoole_tick provides a high accuracy tick service, but only can tick every same micro-seconds or tick atfer some micro-seconds from now.

What I need is something like crontab in Linux, using a format string can easily like "0 0 \* \* mon-fri", and the callback function will called while time's up.

At first I just want to use it as a plugins in [DIServer-framwork](https://github.com/szyhf/DIServer), but after I finished it I find out it's a independent job, so I create this project.

# Difference
As the high accuracy provieded from swoole-extension, not like Linux-crontab, we can set up HIGH LEVEL cron like "\*/20 0 0 \* \* fri", while using 6 params, the format will be "second minute hour day month week"; As 5 params is "minute hour day month week", and it will called on second 1 in that minute like linux-crontab.

# Example
Here is some example I used to test the Ticker, if you find any bugs, please notes me.
```
private $cronString = [
		'* */1 * 3 3 *'          => 'every seconds in Mar 3th.',
		'30 30 21 * * *'         => '21:30:30 on every day.',
		'15,30,45,0 * 23 * * 6'  => 'every 0,15,30,45 seconds in 23 every Saturday.',
		'0 0 */1 * * *'          => 'every hour at 0 minute and 0 seconds.',
		'0 0 4 1 jan *'          => 'every 4:00:00 at January 1st.',
		'* * 7 * * *'            => 'every seconds in 7 o'clock every day.',
		'0-59/2 20 0-23/2 * * *' => 'every 2 seconds in every 20 minutes at every 2 hours in a day.',
		'0 6-12/3 * 2 *'         => 'In February, every 3 hours in 6 o'clock to 12 o'clock.',
		'0 17 * * 1-5'           => 'every 17:00:01 on Monday to Friday.',
		'0 11 4 * mon-wed'       => 'Every 11:00:01 on 4th of a month or in Monday to Wednesday',
		'10 1 * * 6,0'           => 'every 1:10:01 on Saturday and Sunday',
		'45 4 1,10,22 * *'       => '4:45:01 on every 1st,10th,22th in a month.',
		'0,30 18-23 * * *'       => 'every 0,30 minutes in 18-23. Tips:23:30 will called.',
		'*/1 * * * * *'          => 'every seconds',
		'* * * * * *'            => 'every seconds',
		'0 23-7/1 * * *'         => 'equals to "0 23,0-7 * * *"',
	];
```

# Shorthand
While using Month or Week, you can use 'fri' instead of '5' means the friday, the shorthand support was listed below:
```
//Case is ignored
const SHORT_MAP = [
		'sun'       => 0,
		'sunday'    => 0,
		'mon'       => 1,
		'monday'    => 1,
		'tues'      => 2,
		'tue'       => 2,
		'tuesday'   => 2,
		'wed'       => 3,
		'wednesday' => 3,
		'thur'      => 4,
		'thu'       => 4,
		'thursday'  => 4,
		'fri'       => 5,
		'friday'    => 5,
		'sat'       => 6,
		'saturday'  => 6,
		'jan'       => 1,
		'january'   => 1,
		'feb'       => 2,
		'february'  => 2,
		'mar'       => 3,
		'march'     => 4,
		'apr'       => 4,
		'april'     => 4,
		'may'       => 5,
		'jun'       => 6,
		'june'      => 6,
		'jul'       => 7,
		'july'      => 7,
		'aug'       => 8,
		'august'    => 8,
		'sep'       => 9,
		'sept'      => 9,
		'september' => 9,
		'oct'       => 10,
		'october'   => 10,
		'nov'       => 11,
		'november'  => 11,
		'dec'       => 12,
		'december'  => 12
	];
```


