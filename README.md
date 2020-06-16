# clojure-spacemacs-on-android
Setup instructions for Clojure and Spacemacs on Android 


#Spacemacs on Termux

Here is how I got Clojure and Spacemacs running on Android using termux-ubuntu. There might be other ways.

##Notes

###Motivation:
- Code on the go
- Clojure is well suited for coding on smartphones as it is a "dense" language: not a lot of typing required, a lot of code fits on a small screen.

###Good to know
- git works (also magit)
- The Android app Orgzly can be used to edit org mode files and sync them with termux, webdav or dropbox
                                                                                                                                         
###Alternatives:
- Replete (not a complete IDE).
- Nightlight (also via termux-ubuntu, easier to setup, can be used in Android browser so more adapted to the mobile phone experience, But: less functionality, your Clojure program must not crash, otherwise REPL freezes)

###Constraints:
- termux re-uses the Android libc which seems to be incompatible with java, as incredible as this sounds. I had to install termux-ubuntu in order to have java, the openjdk and finally Clojure.
- Installing an Xserver/Desktop environment in termux is possible, but it's really awkward to use. I do not recommend it. Hence we are left with the terminal version of emacs. Impact on spacemacs: no fringe, syntax highlighting only underlines the code (no waves or different colors for underlining possible). As a workaround I activated different background colours for errors and warnings (attention: the set of colours is also limited, there is an emacs command that displays all available colors).
- Currently, I haven't solved a problem in termux-ubuntu which prevents me from using the el package manager. Workaround: install emacs in both termux and termux-ubuntu, run all emacs package installations in termux (it works there). Symlink the .emacs.d folder and the .spacemacs file to the termux-ubuntu subfolders (I know that's ugly).
- None of the standard Android on screen keyboards (OSK) feature special keys like ALT, CTRL, TAB etc. Workaround: 1) install Hacker's Keyboad 2) Configure Extra Key rows in termux https://wiki.termux.com/wiki/Touch_Keyboard . I use a combination of the two. I switch between Hacker's Keyboard and my Samsung OSK via a little icon in the bottom row of the screen (that's configurable on Samsung phones).

###The main ingredients are:
- termux (terminal emulator for Android and some Chromebooks).
- termux-ubuntu (a chroot providing an Ubuntu Terminal environment, do not confuse with ubuntu-in-termux which is less stable)
- optionally termux-widget (costs 3 pounds or so, allows to put a shortcut to a shell script on the Android homescreen. I use it to launch termux-ubuntu, it could be tweaked to launch spacemacs.). 
- Hacker's Keyboard

##Installation:

￼###Install Termux from the Play Store (free)

This is straightforward. Termux has it's own package manager pkg, but if you type apt iso. pkg, that will also work.
It comes with many packages. I have once installed a complete jupyter notebook stack with numpy, scipy etc. 
After the apt update / apt upgrade, I recommend to install fish or tweak bash in order to have some sort of autocompletion.
Termux can read and write the external SD card (you need to run termux-setup-storage), but git can't write to the SD card. So I don't recommend creating any projects on the SD card.

Install a text editor (nano or vi with apt, or micro via curl cf. their website).
Configure the termux extra key rows https://wiki.termux.com/wiki/Touch_Keyboard

￼###Install termux-ubuntu

As I said above, java can't be installed directly in termux. To be completely honest, there is one package which installs java, but it's terribly slow. It's better to install the termux-ubuntu chroot, which is here: 
https://github.com/Neo-Oli/termux-ubuntu
This creates a folder subsystem, and a startup script. The result is a terminal version of ubuntu. The available packages depend on your architecture. I was disappointed to learn that I have a 32 bit architecture, although the processor is 64 bit. 

mkdir -p ~/jails/ubuntu 
cd ~/jails/ubuntu
wget https://raw.githubusercontent.com/Neo-Oli/termux-ubuntu/master/ubuntu.sh 
sed -i 's/disco/eoan/g' ubuntu.sh
bash ubuntu.sh


￼###Install Clojure in termux-ubuntu

Start your termux-ubuntu session (you might want to create a shortcut for that later via termux-widget).

####Dependencies:
apt update 
apt-get install openjdk-8-jdk curl rlwrap git

create hosts file, needed for Cider nrepl:
echo -e '## \n# Host Database\n# \n# localhost is used to configure the loopback interface \n# when the system is booting. Do not change this entry. \n## \n127.0.0.1 localhost \n255.255.255.255 broadcasthost ::1 localhost' > /etc/hosts

####Clojure:
curl -O https://download.clojure.org/install/linux-install-1.10.1.469.sh
chmod +x linux-install-1.10.1.469.sh
./linux-install-1.10.1.469.sh

####Leiningen:
curl -O https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein
mv lein ~/../bin/lein
chmod +x ~/../bin/lein
bash ~/../bin/lein


￼###install spacemacs 

Note: In termux-ubuntu the installation of packages doesn't work, error Bad address and other errors.

Workaround:
- install emacs in both termux and termux-ubuntu
- copy the folder .emacs.d from  termux to termux-ubuntu
- recompile packages

####Install spacemacs in termux:
apt install emacs
git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d 
I recommend to check out the dev version, so you can use clj-kondo

####Install emacs in termux-ubuntu
Do not install the ubuntu emacs package, rather:
apt install software-properties-common
add-apt-repository ppa:kelleyk/emacs
apt install emacs26-nox (the terminal version of emacs)
Again, I recommend to check out the dev version, so you can use clj-kondo.


###Link the two emacs installations
####Copy the folder .emacs.d from  termux to termux-ubuntu
cp .emacs.d jails/ubuntu/ubuntu-fs/root/ -r
cp .spacemacs jails/ubuntu/ubuntu-fs/root/.spacemacs

####Symlink the .emacs.d folder and the .spacemacs file
rm -rf .emacs.d/
ln -s jails/ubuntu/ubuntu-fs/root/.emacs.d/ .emacs.d

rm -rf .spacemacs
ln -s jails/ubuntu/ubuntu-fs/root/.spacemacs  .spacemacs

Now you can update spacemacs in termux, and the spacemacs termux-ubuntu gets automatically updated.
