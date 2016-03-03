# yacrontab-swoole 
This is a php class implement a timetable service using format string like linux crontab, dependence on swoole-extension.

# Quick start
`
<?php
include './CrontabTicker.php'

$crontab = new \CrontabTicker();
$crontab->Crontab('* */20 * * *')
	->Then(function ($userParams)
	{
		echo 'crontab called';
        },
        $userParams);
//The callback function will be called every 20 minitues.

#
