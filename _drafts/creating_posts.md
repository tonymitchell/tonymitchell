---
title: Creating Posts
---

## Getting ready to write

- Check out source in WSL / Linux
    ```console
    git clone git@github.com:tonymitchell/tonymitchell.git
    ```
- Open root folder in VS Code
    ```console
    code .
    ```
- Run `bundle exec jekyll serve  -l --drafts` to run a local server w/LiveReload (-l) and drafts visible
    ```console
    bundle exec jekyll serve  -l --drafts
    ```

## Markdown Syntax references

Markdown:
https://www.markdownguide.org/cheat-sheet/

Syntax highlighting:
https://github.com/rouge-ruby/rouge/blob/master/docs/Languages.md

Jekyll diagrams:
https://github.com/zhustec/jekyll-diagrams



## Jekyll / Chirpy references

Chirpy Theme: https://github.com/cotes2020/jekyll-theme-chirpy
Static Assets: https://github.com/cotes2020/chirpy-static-assets

https://jekyllrb.com/docs/step-by-step/01-setup/
https://jekyllrb.com/docs/posts/
https://chirpy.cotes.page/posts/getting-started/

https://mademistakes.com/mastering-jekyll/how-to-link/

Running a local server: 
- https://chirpy.cotes.page/posts/getting-started/#running-local-server
- https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

Future:
https://docs.github.com/en/get-started/using-git/splitting-a-subfolder-out-into-a-new-repository
https://jekyll.github.io/github-metadata/site.github/







## AVR Reference links:

USB/IP
https://github.com/dorssel/usbipd-win/wiki/WSL-support
https://devblogs.microsoft.com/commandline/connecting-usb-devices-to-wsl/
https://www.kernel.org/doc/html/latest/usb/usbip_protocol.html
USB Device lookup: https://devicehunt.com/view/type/usb/vendor/2341/device/0001
https://gitlab.com/alelec/wsl-usb-gui

VS Code:
https://code.visualstudio.com/docs/editor/tasks


http://jaseemjas.blogspot.com/2013/05/avrdude-with-usbasp-programmer.html
https://www.elecrom.com/avrdude-tutorial-burning-hex-files-using-usbasp-and-avrdude/

Finding serial devices
- https://rfc1149.net/blog/2013/03/05/what-is-the-difference-between-devttyusbx-and-devttyacmx/
- https://unix.stackexchange.com/questions/144029/command-to-determine-ports-of-a-device-like-dev-ttyusb0



Other docs:
https://tinusaur.com/guides/avr-gcc-toolchain/
https://github.com/hexagon5un/AVR-Programming/blob/master/setupProject/Makefile
