> This here answers the question on:
>
> How to use the virtual RTC to keep the time of a VM
>
> This is for VMs which neither have `chrony` nor `systemd` nor similar gigantic monsters
> and just need a small and easy to use tool to keep the time.


# Dancing the Timewarp .. again


I have a VM.  The VM needs to keep correct time.  Of course.

The VM in question is very old.  So there are no monsters like `chrony` which could possibly do the job.

VMs have several properties which non-VMs do not have:

- There is an RTC which always has the correct UTC time because it is synced from the host.
- While they are suspended the internal time stops (but the RTC still is correct)
- CPU cycles may get lost, so the internal clock is highly instable

Usually it is exactly the other way round, the internal clock is very reliable but the RTC has some unpredictable (temperature based etc.) drift.

So all which is needed is:

- A small tool which
- reads the time from the RTC
- and corrects the local time
- using `adjtime` syscall.
- It must do this on a regular basis, say all 10s.
- And if the time is too far off in the past
- it must jump the time forward to the correct time.
- It never must jump the time backwards.
- It must run completely autonomous and reliable.


## Usage

	git clone https://github.com/hilbix/franknfurter.git
	cd franknfurter
	make
	sudo make install

then run

	/usr/local/bin/franknfurter 30 franknfurter.log

where

- `30` warps the local clock forward if it is off more than 30 seconds.
- `/var/tmp/franknfurter.log` is the logfile of frankfurter
  - this redirects stdout and stderr to the given file
  - the logfile is recreated on a regular basis, such that you can easily rotate it

It needs access to `/dev/rtc` and the `adjtime` syscall, hence it probably must run by `root`.

To autostart you can try something like that in `crontab`:

```cron
-* * * * * flock -Fn /var/tmp/franknfurter.lock /usr/local/bin/frankfurter 30 /var/tmp/frankfurter.log
```


## FAQ

WTF why?

- I really have no idea why I was unable to find something like this
- As I was unable to find one, I tried to create one mysself.

franknfurter?

- [Rocky Horror](https://en.wikipedia.org/wiki/The_Rocky_Horror_Picture_Show) of course

How to log to `stdout` instead of a file?

- Use `-` as filename for the 2nd argument

How to log to stdout instead of a file?

- Like `stdout` but use redirection to `>&2`, too

How to use `stdout`/`stderr` as usual?

- Leave away the 2nd argument or leave it empty `''`

License?

- This Works is placed under the terms of the Copyright Less License,  
  see file COPYRIGHT.CLL.  USE AT OWN RISK, ABSOLUTELY NO WARRANTY.
- Free as in free beer, free speech and free baby.

