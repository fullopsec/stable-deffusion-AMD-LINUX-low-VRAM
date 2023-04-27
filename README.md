# stable-deffusion-AMD-LINUX-low-VRAM

Youtube video: https://youtu.be/p2H_Zh4lJTI 

Start hosting your own AI for image generation with stable diffusion. Too much people use google collab and pay outrageous price, now you can do it for free with your GPU.

This video show how to install stable diffusion and make it work with only an 8GB VRAM AMD GPU on Linux. AMD GPU's are working thanks to ROCM. We can use an old 8GB VRAM GPU with the help of some launch options that prevent over-allocation of GPU memory.

Lower VRAM GPU might work but you have to try  your own combination of launch parameters 

Must have: 

Linux OS (I use a debian based distro)

AMD GPU (8GB works)

Each GPU can be different. The parameters I needed might be different from yours, try them. It took me 3 hours to find which worked for my GPU.

My launch parameters:
--no-half --always-batch-cond-uncond --opt-sub-quad-attention --medvram --disable-nan-check

List of launch parameters:
https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Optimizations

VAE I USED (vae-ft-mse-840000-ema-pruned.ckpt):
https://huggingface.co/stabilityai/sd-vae-ft-mse-original/tree/main 

______________

Download the driver for your amd gpu

https://www.amd.com/en/support



Add yourself to the render and video groups using
```
sudo usermod -a -G render yourusername      
sudo usermod -a -G video yourusername
```

Confirm you have python 3 installed by typing into terminal:

`python3 --version` 

**Install rocm**
```
sudo amdgpu-install --usecase=rocm --no-dkms
```  

Reboot

`sudo reboot`


After rebooting, this command should show your gpu:

`rocminfo`

Clone the stable diffusion GUI repo:

```
sudo apt-get install git
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
```  


If you have python 3.8 enter this to make sure you have VENV capabilities (replace with your python version):

  `apt install python3.8-venv`    
  
  
   Install pip3 and wheel and update them
   
```
sudo apt install python3-pip           
python -m pip install --upgrade pip wheel
```      

Download any stable diffusion model you like and put it in models/Stable-diffusion folder

You can go to https://civitai.com/ to find models (also a good source for prompts)

Upgrade to latest stable kernel(for better performances):
```
sudo apt-get update
sudo apt-get dist-upgrade
```

reboot again

`sudo reboot`


Go to the virtual env of SD:
```
cd stable-diffusion-webui
python -m venv venv 
source venv/bin/activate
``` 

Next is installing the PyTorch machine learning library for AMD:

Lots of people were telling to install 5.2 but it didn't work for me. I used 5.4.2

This is to be installed in the VENV, not on the OS!

`pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/rocm5.4.2` 


After that’s installed, check your version numbers with the command
`pip list | grep 'torch'` 

*Torch, torchvision and torchaudio version numbers that come back should have ROCM tagged at the end.*

![image](https://user-images.githubusercontent.com/114147068/231775700-e292b9a3-8969-4018-8a20-6a040e47ec5c.png)



I have a vega 56 8GB so ...

It wasn't enough VRAM to do latent upscale!

But I found a fix!

Optimise Vram usage with:

`--medvram` and `--lowvram` launch arguments 

But it gives reallybad quality oil painting style.(for me)

**You need --always-batch-cond-uncond with lowvram and medvram options to prevent bad quality (for me at least)**

**If your results get black image your card probably dont support float16, use: --precision full (at the cost of more VRAM)**

I benchmarked so many options (list of options is on the top of the page).

I couldn't generate image bigger than 1000x1000 in img2img because of my VRAM (8GB) so I had to benchmark all launch options.

You have to try all possible combinaison to find which allow you to generate the content you need.

These options may(not sure) be different for others graphics card model (mine is vega 56 8GB).

--medvram is enough for a 8GB card. If you have less VRAM use --lowvram

  ## **The options that enabled me to generate nice images(1024x1024) upscaled to (2048x2048) was:**

**--no-half --always-batch-cond-uncond --opt-sub-quad-attention --medvram --disable-nan-check**

`python launch.py --opt-sub-quad-attention --medvram --disable-nan-check --always-batch-cond-uncond --no-half`

Time taken: 1m 16.33s  (1024x1024) img2img

Time taken: 1m 39  at (1024x1024) hires fix

Your base images (512x512px) are quick to generate (10-20s)


if something doesnt work make sure:

`pip list | grep 'torch'`

torch, torchvision and torchaudio version numbers that come back should have ROCM tagged at the end.


Every time you want to launch stable diffusion go back to the venv where all dependencies are installed:
```
cd stable-diffusion-webui
python -m venv venv 
source venv/bin/activate
python launch.py --opt-sub-quad-attention --medvram --disable-nan-check --always-batch-cond-uncond --no-half
```


You might want to watch out for your VRAM usage and system temps. I use:
```
sudo radeontop
watch -n 1 sensors
```

Adjust your fan curve if your temps are too high!!!(70C)



In the next video I will show you how to make them 2048x2048 with high details. (same GPU 8GB)

I will benchmark multiple methods!
