---
title: Fix Unheathy State of Cloudflare Tunnel
date: 2023-04-08
---

If you find the status of your cloudflare tunnel is constantly unhealthy, you
can run check the log of your `cloudflared` process, and if you're runing on
Linux with `systemctl`, you can just do,

```
sudo systemctl status cloudflared
```

and if you found there is `failed to dial to edge with quic` in the error log,
you can refer to this [issue](https://github.com/cloudflare/cloudflared/issues/617) to do this fix like this

* Vim /etc/systemd/system/cloudflared.service file and update the `cloudflared`
    run option with `--protocol http2 --no-autoupdate`
* Run `systemctl daemon-reload`
* Restart the `cloudflared` with `sudo systemctl restart cloudflared`
* Done

