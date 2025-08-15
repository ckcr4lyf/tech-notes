# Transparent Screenlock

Swaylock used to support alpha channels allowing for transparent screenlock, but unfortunately [they removed support for this](https://github.com/swaywm/swaylock/issues/278), since sway  (and wayland) planned to drop input-inhibitor.

TODO: Confirm which version removed it

WaylSwayand dropped it in [1.10](https://github.com/swaywm/sway/releases/tag/1.10) as a result of wlroots dropping input-inhibitor in [0.18.0](https://gitlab.freedesktop.org/wlroots/wlroots/-/tags/0.18.0).

However, if you compile swaylock, sway and wlroots in the version that have alpha & input-inhibitor support, you can still use transparent lock.

## Compilation Instructions

For sway (and wlroots):

```
git clone https://github.com/swaywm/sway.git
git checkout 1.9

git clone https://gitlab.freedesktop.org/wlroots/wlroots.git subprojects/wlroots
cd subprojects/wlroots
git checkout 0.17.4

cd ../../
meson build/
meson configure -Dwerror=false
ninja -C build/
```

For swaylock:

```
git clone https://github.com/swaywm/swaylock.git
git checkout 1.5 (TBD?)
meson build
```

### Swaylock comment
There is a line which causes compilation warnings, here is a patch:

```
$ git diff
diff --git a/main.c b/main.c
index c65d68a..a5d2986 100644
--- a/main.c
+++ b/main.c
@@ -1057,7 +1057,7 @@ static int load_config(char *path, struct swaylock_state *state,
                char *flag = malloc(nread + 3);
                if (flag == NULL) {
                        free(line);
-                       free(config);
+                       // free(config);
                        swaylock_log(LOG_ERROR, "Failed to allocate memory");
                        return 0;
                }
```