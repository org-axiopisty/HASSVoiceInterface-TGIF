---
tags:
  - compute
  - AI
  - NVIDIA
  - Blackwell
  - ARM
  - 10GbE
  - 200Gb
  - 400Gb
  - householdIT
  - performance
  - pricing
  - Asus
  - Spark
  - projectDigits
  - DGX
  - GX10
date: 2025-05-30
---
## New Hotness: Auxiliary Compute

These are exciting times. Cloud services with flagship and foundation models, local AI compute customized merges and remixes off HF launched via Ollama, LMStudio or any other of dozens of options; and a new category of hardware to many of you that will probably become more common in the next five years: purpose-built AI compute modules for desktops like the [NVIDIA DGX Spark](https://www.nvidia.com/en-us/products/workstations/dgx-spark/#overview), which puts 1000 TOPS of AI compute in a cute little box that might scald you if you touch it but what a burn that would be!

Devices like Spark enable you to run 200b models entirely inside with an optimal architecture for this purpose. They're using a similar compute design on ARM with multiple co-processing and three types of compute happening, that reads similar to the way Apple designed their own silicon for processors, and contains at least 128GB of memory. There are NVME slots in there for fast storage and thank g-d it even has 10GbE on the management interface it'll fit just fine with data science teams and data-centers if you're so inclined to rack these suckers. You can link them at 200Gb or 400Gb together and load a 400b Llama 4 on those monsters though!  🤯  Most of us won't have the pleasure, but it's easy to admire the achievement. They will outperform intel and AMD platforms for the same reason Apple Silicon is so damn good at AI: no bus traversal, not until you exit a network interface, at any rate.

If you already have 2-3 RTX GPUs slotted in your 56 core dual Xeon HP-Z, or own a mac Pro or Studio with 128GB+ of memory, this may not sound all that appealing, _especially_ when you hear the pricing, but these devices will become more and more common, prices will drop somewhat, but until it's commonplace and available at a Retailer don't expect them to dip below USD $3000.

The 400Gb interconnect model of Spark is expected to retail over USD $4000, the 200 Gb model should be around 3, but competition is beginning already with Asus joining the chat:

- [DGX Spark](https://www.nvidia.com/en-us/products/workstations/dgx-spark/) is Blackwell architecture, formerly referred to as Project Digits MSRP estimates range from USD $3999 to USD $4999 
- [Asus Ascent GX10](https://www.asus.com/event/asus-ascent-gx10/) -MSRP $2999 and a mini NVIDIA powered PC with 128GB of memory and 200Gb networking for clustering; *also on Blackwell*

Don't forget these don't require your computer running. They have graphics outputs as well so you can stick a head on it if you want, or now you finally have something for HDMI 4 on your TV :shrug: There will be many circumstances where buying a networked  compute module would be far superior to chasing GPUs at inflated prices. 