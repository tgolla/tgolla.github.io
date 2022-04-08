Install Jekyell on windows (https://jekyllrb.com/docs/installation/windows/).

1. Download and install a Ruby+Devkit version from [RubyInstaller Downloads](https://rubyinstaller.org/downloads/). Use default options for installation.

2. Run the ```ridk install``` step on the last stage of the installation wizard. This is needed for installing gems with native extensions. You can find additional information regarding this in the [RubyInstaller Documentation](https://github.com/oneclick/rubyinstaller2#using-the-installer-on-a-target-system). The RubyInstaller should automatically kick off this step in a command window.

3. Open a new command prompt window from the start menu, so that changes to the ```PATH``` environment variable becomes effective. In Visual Studio Code you will need to restart the application for Terminal to pull the new path. Install Jekyll and Bundler using ```gem install jekyll bundler```

4. Check if Jekyll has been installed properly: ```jekyll -v```

5. Run ```bundle update```.

To execute Jekyll run the following command in a cmd terminal.

```
bundle exec jekyll serve --drafts --future
```