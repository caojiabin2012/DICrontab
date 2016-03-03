# DICrontab
This is a php class implement a timetable service using format string like linux crontab, dependence on swoole-extension.

Thanks to the project [swoole-crontab](https://github.com/osgochina/swoole-crontab), I learned how to analyze the crontab string.

DICrontab is not the same as [swoole-crontab](https://github.com/osgochina/swoole-crontab), that's a complete application, and DICrontab is just like a tool, a simple library, a class, you can inclued it and use is in your project.

# Quick start
```
<?php
include './CrontabTicker.php';

$crontab = new \DIServer\Ticker\CrontabTicker();
$crontab->When('* */20 * * *')
	->Then(function ($userParams)
	{
		echo 'crontab called';
		return false;//return false if you want to cancle this cron.
        },
        $userParams);
//The callback function will be called every 20 minitues.

//If you just want to test when will the ticker tick next time
//Use the Next() function instead, Next() will return the Iterator of the time table.
//Also you can use From(startTime) to asume the cron start at what time you need instead of time().

$iterator = $crontab->When('* */20 * * *')
		//->From( mktime(14, 00, 00, 3, 3, 2016) )//Optional
		->Next();

$count = 0;//only print next 10 cron.
foreach($nextTickTime in $iterator)
{
	if($count++ < 10)
		echo $nextTickTime;
	else
		break;
}

/*
Simple of crontab-string
$crontabString :
	 *                       0     1    2    3    4    5
	 *                       *     *    *    *    *    *
	 *                       -     -    -    -    -    -
	 *                       |     |    |    |    |    |
	 *                       |     |    |    |    |    +----- day of week (0 - 6) (Sunday=0)
	 *                       |     |    |    |    +----- month (1 - 12)
	 *                       |     |    |    +------- day of month (1 - 31)
	 *                       |     |    +--------- hour (0 - 23)
	 *                       |     +----------- min (0 - 59)
	 *                       +------------- sec (0-59) (Optional)
You can use 5 params like what crontab use in Linux, and use the 6th param as to descript the SECONDS.
*/
```


# Dependence
+ My develop enverment is CentOS 7.
+ PHP 7.0.3
+ [Swoole-extension](https://github.com/swoole/swoole-src/releases) 1.8.3 
+ I think php5.5+ and swoole 1.7.7+ is Ok, but I can't test it.
+ I used swoole_timer_after, so must at least swoole-1.7.7.
+ It also used keyword 'yield' in php, so 5.5+ is required.

# Notes
As this class is base on swoole_timer_after, please notes below while using in swoole_server or asyns-swoole-client:

1. If the Process was reloaded, the cron tick will be deleted, so if you want add an resident crontab, please record your cron-string and init the ticker on process start, some swoole callback like 'OnWorkerStart', cause the cron-string descripted the timetable, don't worry about lose the ticker, just set a new one.

2. If you want to do something like send 1 message to every user, please notice that every Process can has its own tiker, if you add the tick in OnWorkerStart, asume you have 2 Worker Process and 4 TaskWorker Process and you set the ticker on every worker start, you will have 6 ticker and callback at same time, every client will receive 6 message. So, just use something like "if($server->worker_id==0)" to add only one ticker.

# Description
While working my on swoole-framework [DIServer-framwork](https://github.com/szyhf/DIServer), I find is not easy to implement some timer job like send a message to every clients on every Monday to Friday, swoole_tick provides a high accuracy tick service, but only can tick every same micro-seconds or tick atfer some micro-seconds from now.

What I need is something like crontab in Linux, using a format string can easily like "0 0 \* \* mon-fri", and the callback function will called while time's up.

At first I just want to use it as a plugins in [DIServer-framwork](https://github.com/szyhf/DIServer), but after I finished it I find out it's a independent job, so I create this project.

# Difference
As the high accuracy provieded from swoole-extension, not like Linux-crontab, we can set up HIGH LEVEL cron like "\*/20 0 0 \* \* fri", while using 6 params, the format will be "second minute hour day month week"; As 5 params is "minute hour day month week", and it will called on second 1 in that minute like linux-crontab.

# Example
Here is some example I used to test the Ticker, if you find any bugs, please [notes](https://github.com/szyhf/DICrontab/issues/new) me.
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


