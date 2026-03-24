
# Lenovo Slim 7 Pro X Linux Compatibility and Tweaks
This is a mostly personal repo to store and track various tweaks, tools, and settings I've used to install and optimize Linux on the Lenovo Slim 7 Pro X (14arh7). Also sold under the Ideapad and Yoga brands in some regions.

Lenovo product page:
https://psref.lenovo.com/WDProduct/Lenovo_Slim_7_ProX_14ARH7

This laptop has excellent Linux compatibility. The only component I have not tested was the original MediaTek wifi adapter. I swapped that out for an Intel AX210 shortly after I bought the machine, just to improve connection stability and speed (highly recommended, btw). 

## Distributions
Most mainstream distributions should work. I've tested with Linux Mint and I currently run Fedora 43. Avoid running old kernel versions, as they will likely lack proper hardware support. As an aside, I had much better battery life in Fedora than Mint, so take that as you will.

## Installation Tips
**Dual Booting and Bitlocker**
While my distro of choice, Fedora, does support Secure Boot, I find it to add complexity without much benefit for most regular users. YMMV. 

If your Windows installation is secured with Bitlocker (like most OEM installations are), disabling Secure Boot will lock you out of Windows and require account recovery with Microsoft. **Caution**: Some distributions' live environments will attempt to disable Secure Boot in BIOS for you, which can trigger this lock unexpectedly.

If this applies to you, be sure to disable Bitlocker in Windows prior to proceeding. You can do this in the Windows system settings, it takes about 45 minutes, and it's non-destructive. Once complete, enter BIOS 

Outside of that, installation on this machine is typical and uneventful. Simply boot into a live environment, and follow the installation steps for your distro of choice.

## Nvidia Drivers (Fedora)
Depending on your distribution, it is likely that you'll need (or want) to install the proprietary nvidia drivers to gain full functionality of the RTX 3050 that ships with this computer.

On Fedora, the nvidia driver is packaged and maintained by RPMFusion. Due to Fedora's strict policies, access to this repo is not enabled by default. There are two ways you can add it. **Be sure to perform a full system update prior to proceeding.**

**Adding RPMFusion Repo (GUI)**
In the GNOME Software app, or KDE Discover application, navigate to the settings and look for the repos/sources. In KDE, the immediate Settings panel lists all configured repos and their state. You really only need to enable `RPM Fusion For Fedora 4x - Nonfree - NVIDIA Driver`. You may need to relaunch or refresh your software application after doing this, if it doesn't automatically.

**Adding RPMFusion Repo (CLI)**
Open your Terminal of choice and run the following command:

`sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`

Then
`sudo dnf update -y`
And perform a reboot if any packages were updated, just to be safe.

**Installing Nvidia Drivers (GUI)**
Once the repo is enabled, you should be able to search Software or Discover for `nvidia driver`, and look for `NVIDIA Linux Graphics Driver`. Simply install it like any other app, and reboot.

**Installing Nvidia Drivers (CLI)**
 1. Update the system if you haven't already.
	 2. `sudo dnf update -y`
	 3. Reboot.
 2. Install the drivers.
	 3. `sudo dnf install akmod-nvidia`
	 4. (Optional, CUDA support for video editing etc): `sudo dnf install xorg-x11-drv-nvidia-cuda`

Once the drivers are installed (either way), please wait about **3-5 minutes** for the new driver to build, then reboot.

**Black Screen after Installation**
...Is normal. Any time the drivers are installed or updated, the system must rebuild the driver kmod (kernel module) that plugs into the Linux kernel. This takes a few minutes to complete, so expect your computer to hang on a black screen with the fans running for 2-3 minutes while this process completes. This is normal.

**Verifying**
Running `modinfo -F version nvidia` should return the driver version number if the drivers are installed correctly. Some desktop environments will also show the dedicated GPU by name. If it's identified there as something like "nv112," you are still on the basic nouveau drivers.

**Hybrid Graphics Switching**
Modern distributions and current nvidia graphics drivers do not require any user interaction to offload graphics intensive tasks onto the nvidia dGPU. By default, the desktop environment and most standard applications will run off the integrated AMD GPU.

It is not necessary, and is in fact detrimental, to manually "switch" to the nvidia GPU, as this will force all applications to run on the power-hungry RTX GPU. This would generate excessive heat and noise, and tank battery life.

If you need to force a specific application to run on a specific GPU, most desktop environments will you to do this by their right-click Context or Properties menus. You can also force it by adding the following environment variables to an applications's launch arguments:
`__NV_PRIME_RENDER_OFFLOAD=1`
`__GLX_VENDOR_LIBRARY_NAME=nvidia` (For OpenGL applications)

Again, this should not be necessary, but there ya go.

## Windows Hello (IR Camera Login)
I originally ran the stable version of Howdy, and it miraculously kept working as I updated to Fedora 41 and 42, but it finally broke as of 43 and I haven't bothered setting it up again.

That said, here's another user who did a guide for Fedora 43:
https://github.com/madhavramini/howdy-fedora-43-kde-installation-guide

## Speaker Quality & Loudness
The Slim 7 Pro X has two speakers. While they're not anything to write home about at their best, they are extremely quiet and tinny when driven without any enhancements. Luckily we can fix this with the right impulse file and the Convolver effect in [EasyEffects](https://github.com/wwmm/easyeffects), using an IRS file which helps build a profile of the physical speakers' audio output. And luckily, Lenovo is pretty lazy about this from what I can see, so this isn't that hard.

 1. Install EasyEffects, see the link above for instructions. I found Fedora's version to be out of date, so I switched to the Flatpak from Flathub.
 2. Navigate to the Effects tab on the bottom. 
 3. If there are any effects selected in the top left, remove them. Then click "+Add effect," and add a Convolver effect.
 4. With the Convolver you just added selected, click the "Impulses" button (bottom left of the graph area), and load one of the .irs files available for download from this repo.
 5. Play some music, and see how it sounds.

**Impulse (IRS) Files**
I uploaded a few impulse files to this repo. I said early Lenovo is lazy here, and I say that because I came across half a dozen impulse files for various thin-frame premium Lenovo laptops in my searches, and most of them them appear to be identical. 

I'm currently using the one titled `X13-Gen-1-Music.irs`, and I included Dynamic and Dolby ATMOS as well. These were all IRS files I found on my hunt, but I found the X13 Gen 1 Music preset to be a great fit for the 14arh7. I honestly think it sounds better than it did in Windows.

**Peaking** (Crackling & Distortion)
If you're getting some nasty peaking and distortion, particularly at high volume or with strong lows (bass), try bringing down the Input and Output values on the Convolver effect. I have mine at `-3.0 dB` (Input) and `4.30 dB` (Output). I'm not sure what those do, but it seemed to help.

I would also add another effect called a Limiter. You can add it exactly like you added the first Convolver effect. I have both the input and output on that one set to `-1.20 dB`, and it's been doing great. Be sure the Limiter is the last effect in the chain. It should be, but just make sure.

At this point, even if you don't plan on touching this much again, I would strongly recommend saving your effect settings to a Preset. You can do this with the "Presets" button in the top left. Just give it a name, and click the "+" button next to the name to save it.

**Automatic Preset Loading & Switching**
Once you're happy with the sound, you can leave it. Just set EasyEffects to auto launch on boot, and your config will load with it.

However, this means if you plug in headphones or pair a speaker, it will continue using these effects, which will sound bad. You can set presets to load for specific audio devices, but this presents a problem with headphones still. It's the same audio device, so EasyEffects doesn't know the difference. You could split it to a separate device, but then it won't auto-switch when you plug and unplug your headphones in.

To get around this, we're going to do two things:

 1. From the Presets button on the top, save your preset Locally, then close this Window. 
 2. Go back to the Effects tab at the bottom, and clear all any effects you have. You should basically be left with a "blank" preset.
	 3. We saved your actual preset earlier for this exact reason. Save this one as something like "Bypass" or "Flat
 3. Open the Presets window again (top left).
 4. Click the Autoload tab on the top.
 5. Make sure the laptop's speakers are selected up top, like "Ryzen HD Audio Controller Speaker." 
 6. At the "Local Preset" dropdown, select the one you originally saved that sounds good. 
 7. In the bottom right, click the switch to enable Fallback Preset.
 8. Set the Fallback Preset to the "flat bypass" blank preset you made earlier.

This tells EasyEffects to load in the "flat bypass" preset you made whenever there's no preset defined for a given audio device. For some reason, this system can understand the difference between headphones and speakers, but not the regular autoloader.

The result is: EasyEffects will use the "good" profile whenever you're just using the speakers. For anything else, EasyEffects is bypassed completely.

One more quick note: EasyEffects does add some latency to the audio, so it's probably worth disabling it or setting up bypasses for your DAW or similar if needed. That said, EasyEffects doesn't even seem to see Bitwig when I checked, so I suspect smart DAWs are talking more directly to the sound server anyway.

## Closing Thoughts
tl;dr This laptop is exceptionally well suited to Linux, and Fedora KDE runs like a dream on it.
