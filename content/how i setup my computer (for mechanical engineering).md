# overview

I will divide this article into roughly 2 sections, software/programs I consider essential, and tweaks/workflows I find make me more efficient.

I would highly recommend a windows laptop for engineering students due to the limited availability of certain industry standard software on mac, namely SolidWorks. While it is possible to run [Parallels](https://www.parallels.com/products/desktop/?utm_medium=cpc&utm_source=google&utm_term=parallels&utm_content=&utm_id=43693686&extensionid=&matchtype=e&device=c&devicemodel=&creative=631507300741&network=g&placement=&x-source=ppc&campaign_name=PDfM-B-EN-PS-Tier1-Consolidated&gad_source=1&gad_campaignid=43693686&gbraid=0AAAAAD-bykDbf3b_6qez-jhuOLEyNrtKR&gclid=CjwKCAjw3_PCBhA2EiwAkH_j4nDgCSKKeAE34f1Iu0wNN4I5I7DbXJFcAyME2x4V1UV9nNGOr5KvXBoCztkQAvD_BwE)/[UTM](https://mac.getutm.app/)/similar software on a mac in order to gain a windows environment, if you are in the market for a new laptop near university it is much easier to simply avoid the hassle. I won't go into detail here, but if you want more detail you can see my other post on [[laptop selection]].

# software/apps

There are a lot of different programs I use daily as a mech eng student, some of them mainstream and some of them more obscure, but these are what I consider essential to download on a new machine, and they are further grouped into rough categories. I tend to prefer FLOSS software (Free/Libre and Open Source Software), though efficiency is most important to me and I will use closed and/or paid software if I consider it the best option. I've italicized programs I consider to be "hidden gems" which may not be commonly recommended. I've avoided listing very common/subjective programs like browsers, streaming services, etc. A lot of these work in tandem with the [[apps I use on Android]].

## security

[Backblaze](https://www.backblaze.com/cloud-backup/personal)  
- **Purpose:** 
	- as part of the [[3-2-1 backup method]], I use Backblaze for full cloud computer backup. 
	- it provides *unlimited* storage for anything connected to your laptop/PC, including hard drives. Syncs in the background, and provides peace of mind against theft/damage.
- **Cost:** Paid ($99 USD per year)
- **Platforms:** Windows, Mac

*[Ente Auth](https://ente.io/auth/)*
- **Purpose:** 
	- cross-platform passwordless authentication,, including full desktop support,
	- i.e. you can get 2FA codes on your PC. 
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux, Mobile

[Proton Pass](https://proton.me/pass)  
- **Purpose:** 
	- simple and secure password manager
- **Cost:** Freemium
- **Platforms:** Windows, Mac, Linux, Mobile, Web

[Proton VPN](https://protonvpn.com/)  
- **Purpose:** 
	- VPN for privacy and security  
- **Cost:** Freemium
- **Platforms:** Windows, Mac, Linux, Mobile

---

## engineering

[SolidWorks](https://www.solidworks.com/support/community-download)  
- **Purpose:** 
	- industry-standard CAD for mechanical design  
- **Cost:** Free for students, Paid
- **Platforms:** Windows  

[AutoCAD](https://www.autodesk.com/products/autocad/overview)  
- **Purpose:** 
	- 2D and 3D drafting, widely used in engineering.
	- see my post on AutoCAD setup
- **Cost:** Free for students, Paid  
- **Platforms:** Windows, Mac

[Autodesk Fusion](https://www.autodesk.com/products/fusion-360/overview)  
- **Purpose:** 
	- CNC toolpath generation and modeling
	- I find it to be better/more intuitive for toolpathing than SolidWorks
- **Cost:** Free for students, Paid  
- **Platforms:** Windows, Mac

*[F3D](https://f3d.app/)*
- **Purpose:** 
	- lightweight 3D viewer which **allows the preview of STEP files in File Explorer**
- **Cost:** Free, Open Source  
- **Platforms:** Windows, Mac, Linux

[OMAX Layout and Make](https://www.omax.com/support/software)  
- **Purpose:** 
	- waterjet path layout and cutting
	- [[how to use OMAX waterjet software]]
- **Cost:** Free  
- **Platforms:** Windows  

[Deepnest-Next](https://github.com/deepnest-next/deepnest)  
- **Purpose:** automatic part nesting for sheet cutting  
- **Cost:** Free
- **Platforms:** Windows, Mac, Linux  

[Bambu Studio](https://www.bambulab.com/en/download/bambu-studio)  
- **Purpose:** 
	- slicer for Bambu Labs 3D printers (it just works)
	- see [[guide to 3d printing]]
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux  

[Orca Slicer](https://github.com/SoftFever/OrcaSlicer)  
- **Purpose:** 
	- slicer for all other 3D printers
	- can also work with Bambu Labs, but I find using Bambu Studio gives better results
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux  

[Inkscape](https://inkscape.org/)  
- **Purpose:** vector graphics editor for laser cutting, general CAD prep  
- **Cost:** Free, Open Source  
- **Platforms:** Windows, Mac, Linux  

[STAR-CCM+](https://www.plm.automation.siemens.com/global/en/products/simcenter/STAR-CCM.html)  
- **Purpose:** CFD and multiphysics simulation  
- **Cost:** Paid  
- **Platforms:** Windows, Linux  

[ULS UCP](https://www.ulsinc.com/support/software-downloads)  
- **Purpose:** 
	- laser cutter control software 
	- see [[guide to laser cutting]]
- **Cost:** Free
- **Platforms:** Windows  

---

## documents

[Bluebeam Revu](https://www.bluebeam.com/solutions/revu)  
- **Purpose:** 
	- best PDF software in my opinion, far better than free/paid Adobe Acrobat
	- is an industry standard in engineering consulting for markups
- **Cost:** Free for students  
- **Platforms:** Windows  

---

## messaging

[Beeper](https://www.beeper.com/)  
- **Purpose:** 
	- unified messaging platform  
- **Cost:** Free / Paid  
- **Platforms:** Windows, Mac, Linux, Mobile  

[Spark](https://sparkmailapp.com/)  
- **Purpose:** 
	- unified client with public link generation
	- seamless sync across devices with a single log in
- **Cost:** Free / Paid  
- **Platforms:** Windows, Mac, Mobile  

---

## utilities

[Pandoc](https://pandoc.org/)  
- **Purpose:** 
	- CLI for document format conversion (Markdown, PDF, Word, etc.) 
	- Very useful for converting markdown to word docs.
- **Cost:** Free, Open Source  
- **Platforms:** Windows, Mac, Linux  

[Git](https://git-scm.com/) (+ [TortoiseGit](https://tortoisegit.org/))  
- **Purpose:** 
	- Version control
	- Tortoise provides File Explorer integration, which makes seeing git status super easy 
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux

[SVN](https://subversion.apache.org/) (+ [TortoiseSVN](https://tortoisesvn.net/))  
- **Purpose:** 
	- Version control for CAD/binary files which do not allow diffs/merging. 
	- See my post on using [[version control with CAD]] assemblies.
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux  

[LaTeX](https://www.latex-project.org/) (using [VS Code](https://code.visualstudio.com/) + [LaTeX Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop) + [TeX Live](https://tug.org/texlive/))  
- **Purpose:** 
	- High-quality document typesetting
	- Alternative is using Overleaf, I got fed up with slow render times and lack of easy collaboration with many people on the free plan. I am looking into self-hosting Overleaf.
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux  

[PowerToys](https://github.com/microsoft/PowerToys)  
- **Purpose:** 
	- Productivity tools like PowerToys Run (a launcher similar to macos Spotlight), PowerRename, Color Picker, ZoomIt, etc.
- **Cost:** Free  
- **Platforms:** Windows  

[Quick Share (Google)](https://www.android.com/intl/en_ca/nearby-share/)  
- **Purpose:** 
	- Fast file sharing between nearby Android/Windows devices  
- **Cost:** Free  
- **Platforms:** Windows, Android  

[Tailscale](https://tailscale.com/)  
- **Purpose:** 
	- VPN mesh network for remote access
	- I use it to access some of my self-hosted services across different networks
- **Cost:** Free for personal use  
- **Platforms:** Windows, Mac, Linux, Mobile  

[Parsec](https://parsec.app/)  
- **Purpose:** 
	- Remote desktop and low-latency screen sharing  
- **Cost:** Free / Paid  
- **Platforms:** Windows, Mac, Linux  

---

## productivity

[Toggl Track](https://toggl.com/track/)  
- **Purpose:** 
	- Automatic time tracking on windows, helpful for seeing where your time is spent  
- **Cost:** Freemium  
- **Platforms:** Windows, Mac, Linux, Mobile, Web  

[Freedom](https://freedom.to/)  
- **Purpose:** 
	- Website/app blocker for focus sessions  
- **Cost:** Paid  
- **Platforms:** Windows, Mac, Mobile  

---

## notetaking

[OneNote](https://www.microsoft.com/en-ca/microsoft-365/onenote/digital-note-taking-app)  
- **Purpose:** 
	- General notetaking, synced across devices
	- Best solution for syncing hand-written notes across iPad, Windows, and Android.
- **Cost:** Free  
- **Platforms:** Windows, Mac, Mobile, Web  

[Obsidian](https://obsidian.md/)  
- **Purpose:** Local Markdown-based knowledge management  
- **Cost:** Freemium
- **Platforms:** Windows, Mac, Linux, Mobile  

[Zotero](https://www.zotero.org/)  
- **Purpose:** 
	- Reference manager for academic papers  
- **Cost:** Free  
- **Platforms:** Windows, Mac, Linux, Mobile, Web  
