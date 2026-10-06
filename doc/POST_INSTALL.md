The Freenet node dashboard is available at **https://`__domain__`/** — it is gated behind YunoHost's SSO, so only users of your portal can reach it.

**One last step for good citizenship:** forward port **`__port_peer__` UDP** on your internet box/router to this server. Check the dashboard's network status; with the forward in place peers connect to you directly instead of only through hole-punching.

The node keeps itself current automatically (its supervisor applies official releases; see the admin doc).
