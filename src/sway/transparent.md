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
