# gpu-passthrough-qt
i had to do it to em, copy of the gtk app, but qt5

not based off of https://github.com/uwzis/GPU-Passthrough-Manager ; i just stole his icon violently, and his program no longer works so i made mine (thanks to him all seriousness)

![image](https://github.com/user-attachments/assets/bab17997-c323-4bd8-89d6-c53561702c96)

supposed to pair vga groups and audio groups from lspci and show an output of them

supposed to also work closed source nvidia drivers and open source nvidia drivers

no idea if it works with other distros, i have no idea at all for most of it.

only tested on cachyos / arch. 

dracut = untested |
initramfs = untested |
mkinitcpio = tested |

please write if you have issues

`git clone https://github.com/azurkytheturkey/gpu-passthrough-gtk`

`cd gpu-passthrough-gtk`

`makepkg -si`

![image](https://github.com/user-attachments/assets/966884b9-503e-42c6-ab38-6116dfbcfadd)
