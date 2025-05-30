In general, if you are at all interested in learning about AI, and possibly even adopting an AI for household use to create your own Voice Assistant and beyond,  you may prefer to use local compute capability for _inference_, the "work" that an AI LLM does to generate the output you're requesting. It's better privacy posture as well, and even when I use flagships in the cloud I do embeddings locally on my own compute. You want an appropriate GPU; CPU-only performance is generally lackluster compared to using GPUs or other high-performance purpose-built compute like what some vendors are beginning to announce.

The Friday's Party thread has had some discussion on local AI and hardware requirements, which are hopefully succinctly summarized with the following _Generalizations_ and some key terminology useful in filtering:

- Nvidia RTX-series GPUs with 12-16GB of RAM are an excellent way to not have an agonizing learning experience for Windows 11 and Linux users
- AMD ROCm is making improvements, but not recommended for casual or entry-level users due to less adoption due to AMD just not having the benefit for years of data scientists using CUDA. An AMD desktop with integrated graphics and an NVIDIA GPU is a way many people go and seem happy with; plus since Xeons don't usually have integrated graphics at all not even bad graphics like Iris - so you don't get QuickSync accelerated video encoding on those either but you do on a core i-5/7/9 and AMD has similar features for encoding video
- If you're buying a new mac, BTO as much memory as you possibly can for maximum smug

#### Windows 11 Users
The experience of using an RTX-series [[GPU]] from Nvidia on x64 Windows is first-class. Most software written for AI not being done at the flagship foundation models (OpenAI's o1/o3/o4, gpt-4.1, claude sonnet, Gemini) will assume by default that you have an NVIDIA GPU that you'll be using, or CPU-only scenarios that you don't want because they suck and are slow.

##### Nvidia 
An RTX-30, 40, or 50-series GPU with as much memory as you can afford for Windows 11 users. The newer AI PCs that are becoming available are able to do some local inference for helper applications or lightweight automation and to act as a bridge to file managers, applications and not run simulations or generate high-quality code. NVIDIA and CUDA are the default starting point for the vast majority of what is commonly used today. Docker with NVIDIA GPUs on Windows is a piece of cake. Ollama as well, although you will likely need to add system environment variables for Ollama to listen on the interfaces you need it to otherwise you're trapped in loopback.

##### AMD
If you are an AMD enthusiast and know exactly what you're doing and you are comfortable with the pace AMD has on making their GPUs more popular for AI by all means give it hell and be a trailblazer cuz you might have noticed that there isn't a lot of support for AMD hardware when it comes to LLMs and compute performance. There's probably a lot of room for optimization and it may be realized within months.

Docker makes trying out software for AI a snap on Windows and please do yourself a huge favor and ensure you're using WSL2 and have an Ubuntu instance ready to roll. 

#### Linux Users
I have run AI applications on a variety of Linux environments and similar to the Windows 11 scenario most of what you will be learning assumes you are using an NVIDIA GPU, there are more projects and software for linux AI on AMD/ROCm than I've seen on Windows. LMStudio, Ollama, Msty, Witsy, OpenWebUI, text-generation-ui etc all work great on linux either installed on the platform or deployed via docker.

The atomic distributions like Bazzite are really impressive and a radically new model of operating systems that is more easily understood by software engineers than consumers. If you're a linux enthusiast yes but if you're a casual, I'd stick to Ubuntu, Arch, or Fedora.

Which brings us to our surprise curveball: one more thing.

#### macOS users
I want to preface this by saying I know people have strong opinions on this matter but it's worth mentioning that if you are already a mac user and happily using macOS (since 1984 personally) you may not realize that Apple Silicon is absolutely built for AI as it is used today. Apple's architecture for their M-series in-house compute is a lay-up for everything you want to learn about AI for a couple of reasons that might not be obvious as a newcomer.

Apple Silicon has a radically different architecture than an intel/AMD PC. they were extremely similar through the intel era (so much so two of @emory's 'macs' are not made by apple, and i run a virtualized macOS iCloud Content Cache via my household hypervisor (proxmox on E5 xeons)), but inside every M-series CPU are performance cores, neural cores, efficiency cores, and: gpu cores. It's all inside the same silicon! A call to the CPUs and GPUs do not even traverse a PCI bus. And the memory is a unified memory architecture that is just flat-out entirely available to the entire computer which means when you get a mac with 192GB of RAM you can immediately start running 70b-100b and larger models right on your desk.

YES: you can get faster token generation! And YES, you can easily crush a mac Studio M3 Ultra at inference and token generation if you want to, but you're going to need about $6k-$9k more than I would have spent, but there are valid reasons for going that way if you are adamant about staying on intel. I don't enjoy pcpartpicking and bidding wars, but people that are good at finding those deals find it gratifying too, I won't stop you.

You cannot use an Nvidia or AMD GPU on a mac using Apple Silicon. NVidia hasn't shipped drivers for macOS in years and AMD's are included in macOS.

But here's the smug part; any money you think you were "taxed" by buying Apple products gets fully refunded with interest the moment you buy an Apple Silicon mac with at least 128GB of memory. And if you have an M2 Pro Mini with 16GB RAM and have never played with AI before, you can download LMStudio for macOS and immediately have a conversation with an AI chat bot running entirely on your computer in real-time. Unified memory is unified memory. And Windows 11 has some shared memory capability but it does not perform nearly as well as on macOS in terms of using GPU compute that also has to move memory contents from CPU to Card and back again over a bus.

The mac Pro and Studio should be considered a workstation-class computer, treat them like an HP Z-series or System 76's AI workstations when doing pricing comparisons. 