# Screenshots in record time - up to 2.5x faster than MSS (Multiple Screen Shots)

The module provides classes for capturing screenshots in Windows operating systems. It offers four different classes: ScreenshotOfRegion, ScreenshotOfOneMonitor, ScreenshotOfAllMonitors, and ScreenshotOfWindow. These classes allow you to capture screenshots of specific regions, individual monitors, all monitors, or specific windows (even background windows) on the screen.

For users who need to capture screenshots programmatically in their Python applications, this module can be of great interest. It provides a convenient and efficient way to capture screenshots using the Windows API, allowing for customization and flexibility in screenshot capturing.

One notable advantage of this module is its speed. It utilizes direct access to the Windows API and minimizes overhead, resulting in faster screenshot capturing compared to other libraries like MSS (Multiple Screen Shots). This can be beneficial for applications that require real-time or high-frequency screenshot capturing.

Furthermore, the module has minimal dependencies, making it lightweight and easy to integrate into existing projects. It relies on standard Windows libraries and ctypes for accessing the Windows API functions, reducing the need for additional external dependencies.


## pip install fast-ctypes-screenshots

#### Tested against Windows 10 / Python 3.10 / Anaconda


### How to use it 

```python

from time import time

import cv2
import numpy as np
from fast_ctypes_screenshots import (
    ScreenshotOfRegion,
    ScreenshotOfOneMonitor,
    ScreenshotOfAllMonitors,
    ScreenshotOfWindow,
)


# a simple benchmark function
def show_screenshot(
    screenshotiter, stop_at_frame=100, quitkey="q", show_screenshot=True
):
    def show_screenshotx():
        cv2.imshow("test", screenshot)
        if cv2.waitKey(1) & 0xFF == ord(quitkey):
            cv2.destroyAllWindows()
            return False
        return True

    framecounter = 0
    fps = 0
    start_time = time()
    for screenshot in screenshotiter:
        if stop_at_frame:
            if framecounter > stop_at_frame:
                break
            framecounter += 1
        if show_screenshot:
            sho = show_screenshotx()
        else:
            sho = True
        fps += 1
        if not sho:
            break
    print(f"fast_ctypes_screenshots: {fps / (time() - start_time)}")
    cv2.destroyAllWindows()


if __name__ == "__main__":
    # Take one screenshot

    # ScreenshotOfWindow works even with background windows
    with ScreenshotOfWindow(
        hwnd=920542, client=False, ascontiguousarray=False
    ) as screenshots_window:
        img1 = screenshots_window.screenshot_window()
    print(img1.shape)

    with ScreenshotOfWindow(
        hwnd=920542, client=True, ascontiguousarray=False
    ) as screenshots_window:
        img2 = screenshots_window.screenshot_window()
    print(img2.shape)

    # Screenshot of all monitors
    with ScreenshotOfAllMonitors(ascontiguousarray=False) as screenshots_all_monitor:
        img4 = screenshots_all_monitor.screenshot_monitors()
    print(img4.shape)

    # Screenshot of one monitor, starting at 0
    with ScreenshotOfOneMonitor(
        monitor=0, ascontiguousarray=False
    ) as screenshots_monitor:
        img5 = screenshots_monitor.screenshot_one_monitor()
    print(img5.shape)

    # x0=0, y0=40, x1=800, y1=680 -> rectangle
    with ScreenshotOfRegion(
        x0=0, y0=40, x1=800, y1=680, ascontiguousarray=False
    ) as screenshots_region:
        img6 = screenshots_region.screenshot_region()
    print(img6.shape)
    #######################################################################
    # Take many screenshots
    # from a window
    # screenshots_window is iterable
    with ScreenshotOfWindow(
        hwnd=920542, client=False, ascontiguousarray=False
    ) as screenshots_window:
        show_screenshot(
            screenshots_window, stop_at_frame=1500, quitkey="q", show_screenshot=True
        )

    with ScreenshotOfWindow(
        hwnd=920542, client=True, ascontiguousarray=False
    ) as screenshots_window:
        show_screenshot(
            screenshots_window, stop_at_frame=1500, quitkey="q", show_screenshot=True
        )

    # from all monitors
    # screenshots_all_monitor is iterable too
    with ScreenshotOfAllMonitors(ascontiguousarray=False) as screenshots_all_monitor:
        show_screenshot(
            screenshots_all_monitor,
            stop_at_frame=150,
            quitkey="q",
            show_screenshot=True,
        )

    # from one monitor
    # screenshots_monitor is iterable as well
    with ScreenshotOfOneMonitor(
        monitor=0, ascontiguousarray=False
    ) as screenshots_monitor:
        show_screenshot(
            screenshots_monitor, stop_at_frame=150, quitkey="q", show_screenshot=True
        )
    with ScreenshotOfRegion(
        x0=0, y0=40, x1=800, y1=680, ascontiguousarray=False
    ) as screenshots_region:
        show_screenshot(
            screenshots_region, stop_at_frame=150, quitkey="q", show_screenshot=True
        )

```

### Benchmark - MSS vs. fast-ctypes-screenshots

Benchmark | FPS One Screen - with cv2.imshow - MSS | FPS One Screen - with cv2.imshow - fast_ctypes_screenshots | FPS One Screen - without cv2.imshow - MSS | FPS One Screen - without cv2.imshow - fast_ctypes_screenshots
--------- | -------------------------------------- | -------------------------------------------------------- | ---------------------------------------- | --------------------------------------------------------------
1 screen - 1920x1080 | 14.570637829934096 | 29.75346366655896 | 29.040460293245385 | 32.07449524721684
2 screens - 3840x1080 | 10.885839644361466 | 18.843871526097328 | 17.19836897143781 | 24.268109622214084
region - 2800x960 | 13.54247110293269 | 28.722819983256283 | 21.342393892806598 | 29.528899151991656




```python
# MSS https://python-mss.readthedocs.io/ vs. fast_ctypes_screenshots


# The code for the benchmark 

import mss

print("\n\n-------------------------Benchmark - 1 screen - 1920x1080")
fps = 0
last_time = time()

with mss.mss() as sct:
    for _ in range(150):
        sct_img = sct.grab(sct.monitors[1])
        img = np.array(sct_img)
        cv2.imshow("test", img)
        fps += 1
        if cv2.waitKey(25) & 0xFF == ord("q"):
            cv2.destroyAllWindows()
            break
print(
    f"\n\n\nFPS One Screen - with cv2.imshow - MSS: {fps / (time() - last_time)}\n"
)
print("FPS One Screen - with cv2.imshow - ", end=" ")
cv2.destroyAllWindows()

with ScreenshotOfOneMonitor(
    monitor=0, ascontiguousarray=False
) as screenshots_monitor:
    show_screenshot(
        screenshots_monitor, stop_at_frame=150, quitkey="q", show_screenshot=True
    )
cv2.destroyAllWindows()
#########################################################
fps = 0
last_time = time()

with mss.mss() as sct:
    for _ in range(150):
        sct_img = sct.grab(sct.monitors[1])
        img = np.array(sct_img)
        fps += 1

print(
    f"\n\n\nFPS One Screen - without cv2.imshow - MSS: {fps / (time() - last_time)}\n"
)
print("FPS One Screen - without cv2.imshow - ", end=" ")

with ScreenshotOfOneMonitor(
    monitor=0, ascontiguousarray=False
) as screenshots_monitor:
    show_screenshot(
        screenshots_monitor, stop_at_frame=150, quitkey="q", show_screenshot=False
    )
cv2.destroyAllWindows()
#########################################################
print("\n\n-------------------------Benchmark - 2 screens - 3840x1080")
fps = 0
last_time = time()

with mss.mss() as sct:
    for _ in range(100):
        sct_img = sct.grab(sct.monitors[0])
        img = np.array(sct_img)
        cv2.imshow("test", img)
        fps += 1
        if cv2.waitKey(25) & 0xFF == ord("q"):
            cv2.destroyAllWindows()
            break

print(
    f"\n\n\nFPS All screens - with cv2.imshow - MSS: {fps / (time() - last_time)}\n"
)
cv2.destroyAllWindows()
print("FPS All screens - with cv2.imshow - ", end=" ")
with ScreenshotOfAllMonitors(ascontiguousarray=False) as screenshots_region:
    show_screenshot(
        screenshots_region, stop_at_frame=100, quitkey="q", show_screenshot=True
    )
cv2.destroyAllWindows()

#########################################################
fps = 0
last_time = time()
with mss.mss() as sct:
    for _ in range(100):
        sct_img = sct.grab(sct.monitors[0])
        img = np.array(sct_img)
        fps += 1
print(
    f"\n\n\nFPS All screens - without cv2.imshow - MSS: {fps / (time() - last_time)}\n"
)
print("FPS All screens - without cv2.imshow - ", end=" ")
with ScreenshotOfAllMonitors(ascontiguousarray=False) as screenshots_region:
    show_screenshot(
        screenshots_region, stop_at_frame=100, quitkey="q", show_screenshot=False
    )

#########################################################

print("\n\n-------------------------Benchmark - region - 2800x960")

fps = 0
last_time = time()
monitor = {"top": 40, "left": 0, "width": 2800, "height": 960}

with mss.mss() as sct:
    for _ in range(150):
        img = np.array(sct.grab(monitor))
        cv2.imshow("test", img)

        fps += 1
        if cv2.waitKey(25) & 0xFF == ord("q"):
            cv2.destroyAllWindows()
            break

print(f"\n\n\nFPS Region - with cv2.imshow - MSS: {fps / (time() - last_time)}\n")
print("FPS Region - with cv2.imshow - ", end=" ")
cv2.destroyAllWindows()

with ScreenshotOfRegion(
    x0=0, y0=40, x1=2800, y1=1000, ascontiguousarray=False
) as screenshots_region:
    show_screenshot(
        screenshots_region, stop_at_frame=150, quitkey="q", show_screenshot=True
    )

cv2.destroyAllWindows()

#########################################################

fps = 0
last_time = time()
monitor = {"top": 40, "left": 0, "width": 2800, "height": 960}

with mss.mss() as sct:
    for _ in range(150):
        img = np.array(sct.grab(monitor))
        fps += 1

print(
    f"\n\n\nFPS Region - without cv2.imshow - MSS: {fps / (time() - last_time)}\n"
)
print("FPS Region - without cv2.imshow: ", end=" ")
with ScreenshotOfRegion(
    x0=0, y0=40, x1=2800, y1=1000, ascontiguousarray=False
) as screenshots_region:
    show_screenshot(
        screenshots_region, stop_at_frame=150, quitkey="q", show_screenshot=False
    )



```