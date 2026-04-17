---
title: "World Mouse: Exploring Interactions with a Cross-Reality Cursor"
author:
  - "Esen K. Tütüncü"
  - "Mar Gonzalez-Franco"
  - "Khushman Patel"
  - "Eric J. Gonzalez"
source: "https://arxiv.org/html/2603.10984v1"
created: 2026-04-17
tags:
  - xr
  - interaction
  - input-devices
  - cursor
  - cross-reality
  - scene-understanding
  - semantic-segmentation
  - google
status: true
ingested: 2026-04-17
---
Esen K. Tütüncü [0000-0002-0050-0908](https://orcid.org/0000-0002-0050-0908 "ORCID identifier") Institute of Neurosciences  
of the University of BarcelonaBarcelonaSpain, Mar Gonzalez-Franco GoogleSeattleUSA, Khushman Patel GoogleMountain ViewUSA and Eric J. Gonzalez GoogleSeattleUSA

(2026)

###### Abstract.

As Extended Reality (XR) systems increasingly map and understand the physical world, interacting with these blended representations remains challenging. The current push for “natural” inputs has its trade-offs: touch is limited by human reach and fatigue, while gaze often lacks the precision for fine interaction. To bridge this gap, we introduce World Mouse, a cross-reality cursor that reinterprets the familiar 2D desktop mouse for complex 3D scenes. The system is driven by two core mechanisms: within-object interaction, which uses surface normals for precise cursor placement, and between-object navigation, which leverages interpolation to traverse empty space. Unlike previous virtual-only approaches, World Mouse leverages semantic segmentation and mesh reconstruction to treat physical objects as interactive surfaces. Through a series of prototypes—including object manipulation and screen-to-world transitions—we illustrate how cross-reality cursors may enable seamless interactions across real and virtual environments.

Virtual Reality, Augmented Reality, XR, Interaction, Mouse

![Refer to caption](https://arxiv.org/html/2603.10984v1/Figures/wm-teaser2.png)

Figure 1. World Mouse combines virtual content with a semantically labeled real-world mesh, enabling seamless cross-reality cursor-based interactions such as: 3D selection and manipulation, object surface navigation, and drag-and-drop.

## 1\. Introduction

While XR interactions often emphasize hand ray or mid-air gestures [^4] [^53], these techniques can induce fatigue over time and lack the subtle control experienced mouse users rely on [^38] [^51]. This is partially due to XR’s heavy reliance on direct inputs [^20]. Even more advanced hybrid inputs such as gaze+pinch [^46] still demand physical effort through bimanual interactions [^37], preshaping [^40] or mid-air gesturing [^31] [^56]. In contrast, the mouse’s traditional strengths remain compelling [^15]: many immersive applications would benefit from precise, indirect input–especially those involving long work sessions or detailed tasks like 3D modeling and data annotation [^25] [^48] [^6] [^51]. Moreover, since many XR hardware setups already permit seated or desk-based use, the mouse is well-positioned to serve as a high-performance spatial input, provided the complexity of 3D depth can be addressed. Towards this, previous research has explored mapping the desktop and mouse to the virtual environment [^11] [^16] or directly integrating the mouse to virtual modeling [^57] [^6].

We advance cursor-based XR interaction by adapting the desktop mouse for blended cross-reality environments (i.e., containing both real and virtual interactable elements, see Figure 1). Our approach, which we call the *World Mouse*, leverages a blended scene graph to map 2D mouse movements into 3D interactions across spatially distributed objects, environments, and interfaces. We build upon two core mechanisms. This approach rely on two core mechanisms: First, within-object interaction follows the logic of rasterization to precisely track the mouse’s 2D movements across the 3D surface of a single object [^7]. Second, between-object navigation enables fluid movement across disjoint elements by leveraging an “invisible mesh” that interpolates between nearby objects [^5]. Together, these techniques allow users to effortlessly navigate objects of varying scales and depths, mirroring the simplicity of a desktop cursor but applied to the physical world.

In this paper, we outline the conceptual foundation and technical implementation of the World Mouse, detailing how it dynamically infers and adapts cursor depth in real time. Through a series of prototypes, we demonstrate how this approach provides the precise, familiar control of a desktop mouse while avoiding the fatigue of mid-air gestures. Beyond purely virtual environments, we show how the World Mouse integrates with passthrough augmented reality, enabling seamless interactions across both digital and physical objects in a blended scene.. Ultimately, by augmenting a universally understood device with robust 3D capabilities, we expand the reach of traditional workflows and invite a reimagined role for classic 2D input in spatial computing.

## 2\. Background & Related Work

We contextualize our work by highlighting three key trajectories in spatial input: the evolution of indirect precision input, the limitations of embodied XR techniques, and recent efforts to extend cursor-based interaction into immersive systems.

### 2.1. Precision in HCI

The pursuit of precision through indirect input underpinned much of early human-computer interaction [^26]. The adoption of the mouse in early graphical operating systems of course marked a major leap in usability [^15]. Simultaneously, early explorations into “natural” interactions involving gestures and voice emerged [^8], even though the hardware of the time could not fully support researchers’ vision. The advancement of computing and sensing capabilities (e.g. via commodity IMUs), however, gradually laid the groundwork for modern motion-based interfaces [^28].

As direct touch interactions became ubiquitous in the 2000s [^27], they introduced a fundamental tradeoff: while touch fosters immediate and intuitive actions, it inherently sacrifices precision [^29]. Although styli can restore accuracy [^54], pen-based input has largely remained a niche solution. As digital displays scaled up, novel solutions addressed the challenge of distant interaction, ranging from the explicit transitions between direct and relative input, as in HybridPointing [^17], to pen+hand manipulations for massive displays like WritLarge [^55]. This historical near-versus-far dilemma mirrors current challenges in spatial computing, where interaction is typically framed as either direct touch or distant raycasting [^20].

### 2.2. XR Interaction Techniques

XR interaction has evolved from early controller-based inputs [^3] to incorporate full hand and body tracking [^21], alongside more recent voice and gaze-based techniques [^45]. Historically, immersive experiences favored near-field interactions (0.5–3.5 meters) to simplify depth perception and minimize errors [^58]. However, as spatial computing advances, there is a need for adaptable input techniques capable of handling diverse interaction ranges.

Still, embodied mid-air input can cause user fatigue [^25] [^42] and may lack the accuracy required for delicate tasks. To alleviate this, researchers have explored clutching or altering control-display (CD) gains in XR, such as in Go-Go [^47], Erg-O [^41], and SnapMove [^12]. Extensive surveys and evaluations of 3D object selection techniques [^4] [^36] reveal that few incorporate adaptive pointing [^34] or special CD gains, such as PRISM [^18]. Critically, most systems still heavily rely on raycasting; a study of consumer VR games found that approximately 60% use controller-based rays for selection, vastly outpacing direct hand interaction [^53].

To improve far-distance accuracy and reduce full-arm exertion, recent work has explored micro-gestures [^44], gaze-plus-pinch techniques [^45], and pinpointing [^35]. Other research circumvents the problem by repurposing everyday devices–such as smartphones [^60] [^59] or headphones [^43] –as convenient input proxies. Supported by cross-device toolkits like XDTK [^19], this also facilitates a shift toward multi-device experiences where users can interact with shared spatial content even without dedicated XR hardware [^33].

![Refer to caption](https://arxiv.org/html/2603.10984v1/Figures/function3.png)

Figure 2. Cursor behaviors explored through the World Mouse. Left (Upper): Between Objects Navigation leverages interpolation allowing the cursor to traverse through the space between objects. Left (Lower): Within Object Navigation allows the cursor to move along the surface of any object. Right: 2D to 3D Cursor Transition enables a smooth visual transition when using moving from 2D panels like browsers to a 3D objects.

### 2.3. Cursor-based Interactions in XR

Efforts to enhance precision and reduce fatigue have prompted researchers to revisit cursor-based methods for immersive systems. For instance, Hyve-3D [^14] introduced a 3D cursor for precise collaborative design, demonstrating that minimizing full-arm extensions improves user performance. Similarly, Sun et al. [^49] found that the desktop mouse can outperform hand tracking and trackpads in VR, particularly during seated work on stable surfaces.

Recent work has refined these insights. The In-Depth Mouse [^57] introduced a depth-adaptive cursor in VR, while the Everywhere Cursor [^30] demonstrated that adapting 2D input devices for spatial AR can boost both familiarity and precision. These efforts demonstrate the viability of cursor-based interaction in immersive environments, but they largely address interactions within a single modality (VR or AR). In contrast, we investigate how a depth-adaptive cursor can operate across fully blended environments. Building on these works, the World Mouse introduces an interaction model that seamlessly traverses virtual content and the physical world, supporting flexible, cross-reality workflows.

## 3\. World Mouse Design and Implementation

The World Mouse is an indirect input method designed to provide high precision when interacting with both real and virtual objects. By mapping the standard 2D computer mouse into a blended 3D environment, the system allows users to manipulate spatial assets, navigate complex scenes, and seamlessly interact with digital and analog elements alike.

### 3.1. Bringing the 2D Mouse into 3D Space

We build directly upon the Depth-Adaptive Cursor introduced by the In-Depth Mouse [^57], which enables mouse interaction in VR by casting a ray from the user’s viewpoint and utilizing Voronoi-based interpolation to infer depth in empty space. Specifically, the system maps standard 2D mouse deltas to angular displacements of the cursor ray, converting planar movement into spherical rotation relative to the user’s viewpoint. While their approach explores cursor-based manipulation in purely virtual scenes, we extend this logic to fully blended cross-reality environments.

Rather than relying on discrete screen-space segmentation, World Mouse treats the physical and virtual world as a unified, continuous interaction space. By leveraging a geometric reconstruction of the physical environment (e.g., via Meta’s Scene API or Android XR’s Scene Meshing), we can generate a blended scene graph linking the surfaces of both real-world geometry and virtual assets – effectively acting as an “invisible mesh” connecting the two. This allows the cursor to fluidly traverse the scene, dynamically adjusting its depth based on proximity to physical and digital objects alike. We detail this implementation through three distinct cursor behaviors: navigation within object surfaces, interpolation between objects, and the transition between 2D and 3D spaces.

#### 3.1.3. 2D to 3D Cursor Transition

In XR, users may need to repeatedly switch between interactions with 2D windows and 3D content. Previous work such as HandOver [^51] has explored the transition between standard 2D mouse usage and 3D interaction. With World Mouse, we extend this to explore how cursors may transition when moving from a 2D interactive panel out into 3D space, including the user’s surroundings (Fig. 2). For example, a user might finish retouching an image in a 2D editing application and then slide the cursor out of the application window into the 3D canvas to place the edited image as a poster on a physical wall.

### 3.2. Bringing the Mouse into the Real World

To use a 3D mouse in the real world, we need a robust understanding of the real-world geometry such that the mouse can navigate between real-world elements. For true interactivity, this requires object detection and segmentation, with extra consideration for regions that may be relevant for 2D interactables such as virtual windows, texts, and screens. Recent advances in computer vision have improved understanding of objects and 3D world structure, which we can leverage to provide a basic surface for mouse navigation. The system constructs a mesh from the convex hulls of these detected objects relative to the user’s camera view. Designed for stability, this surface accommodates both static and dynamic objects without requiring expensive recalculation for every viewpoint shift. This results in a continuous interactable surface, unlike the discrete zones in VR, allowing for precise mouse usage within 2D contexts like physical monitors or virtual desktop windows.

![Refer to caption](https://arxiv.org/html/2603.10984v1/Figures/applications.png)

Figure 3. World Mouse Application Scenarios. (Left) Spatial Authoring: Users can spawn virtual objects and manipulate them using high-precision 3D gizmos, including spline editing and vertex-snapping to real-world meshes. (Middle) Cross-Device & IoT Control: Digital content can be transitioned from physical screens into the environment, while IoT devices are controlled via interactive proxies and re-rendered reality filters. (Right) Social XR: The system supports collaborative workflows allowing multiple users to interact with shared virtual assets.

## 4\. Prototype Explorations

To demonstrate how the World Mouse translates into tangible, everyday workflows, we developed a series of prototype interactions (Figure 3). Each illustrates a different aspect of precision input in a mixed environment, from simple selection and manipulation to cross-device collaboration and IoT control.

### 4.1. The Desktop Metaphor in 3D Space

We first explore how the established metaphors of desktop computing – including point-and-click selection, hover-state feedback, and context-sensitive menus – might map to interactions with a blended 3D environment. To support these paradigms, the cursor continuously aligns with environment geometry and provides visual highlights for interactive elements similar to match standard mouse interactions.

Spatial Clipboard (Select, Copy, and Paste): Users can click to select virtual objects or areas in the passthrough view and take screenshots of the current augmented reality scene, capturing both virtual and real-world elements. 3D models from the physical world (scanned via the XR headset) can also be pasted it into the virtual environment. This interaction would import real-world objects into the digital space, creating a seamless blend of physical and virtual.

Right-Click and Contextual Menus: Right-clicking triggers radial context menus that appear around the 3D cursor. These menus “fan out” options such as “Properties,” “Copy,” or “Spawn Object,” with contents that adapt based on the semantic understanding of the target object. This allows users to quickly switch modes or access settings without navigating distant floating UIs.

Scroll for Zoom and Depth: In 3D space, the mouse scroll-wheel can offer valuable continuous input and disambiguation. For example, users can use the scroll-wheel to fluidly manipulate the depth of dragged objects, adjust the scale of virtual objects, or adjust the zoom of magnified views in real space.

### 4.2. 3D Authoring and Manipulation

Beyond typical 2D interactions, the World Mouse enables high-precision spatial creation tools that are typically fatigue-inducing when using mid-air gestures alone.

Spawning and Anchoring Objects: The user can bring up a palette to select objects or 3D widgets to place into the environment. Upon selection, a “ghost” version of the new object attaches to the cursor, letting the user choose exactly where to drop it in physical or virtual space. For example, spawning a virtual note or sticky label to annotate meeting notes on a real white board.

3-Axis Gizmo and Drag & Drop: For CAD-like control, a 3D transform gizmo provides $x,y,z$ handles for fine-grained manipulation. Objects can be moved freely through space or assigned mass to follow physics, sliding across surfaces and balancing on physical planes. The precision is high enough to allow spline editing in 3D space or attaching vertices to real-world meshes.

### 4.3. Cross-Reality and Ambient Interaction

The World Mouse can also bridge digital content with physical environments through connected devices and scene understanding.

Virtual Window: The 3D cursor can manipulate 2D panels to place them within the real-world mesh. This decoupling from physical monitors allows for expanded, ergonomic viewing angles while maintaining application continuity, regardless of where the compute happens.

IoT interactivity: By leveraging semantic labels users can assign interactive proxies to physical objects. Hovering or clicking these proxies allows for direct control of real devices, such as toggling lights or adjusting thermostats. Some interactions may trigger a ”re-rendered reality,” such as using a passthrough filter to simulate turning off lights.

### 4.4. Cross-Device and Multi-User Reality

The integration of personal devices like smartphones as World Mouse controllers in XR environments offers a compelling solution for high-precision input in XR environments. By using precise touchscreens as trackpads for cursor control, these devices provide a level of accuracy and familiarity that often surpasses contemporary hand-tracking or controller-based raycasting (Fig. 4).

![Refer to caption](https://arxiv.org/html/2603.10984v1/Figures/multidevice.png)

Figure 4. Controlling the World Mouse using the touchscreen of a smartphone (left) or smartwatch (right) via XDTK 19

Future iterations could support multi-user blended reality, facilitating collaboration across co-located [^33] and remote setups [^52] [^24]. Because the cursor interaction model is backward compatible, users on standard laptops or non-immersive displays can also participate by interacting with a 2D projection of the shared scene graph. This enables cross-device engagement comparable to multiplayer gaming [^33] or distributed productivity [^10], perhaps even enhanced by proxemics and not just by cursors [^39] [^22] [^23].

### 4.5. Interacting with AI

Recent advances in depth sensing and semantic segmentation [^50] [^32] have expanded the world-understanding capabilities of XR systems from simple passthrough video feeds to semantically-aware environment reconstructions. This context is essential for interactions with AI agents, which require clear deictic references–knowing exactly which object a user is discussing–to function effectively [^9]. While systems like XR-Objects [^13] and LightAnchors [^2] demonstrate the potential of attaching digital information to physical items, relying solely on voice or mid-air gestures to select these targets often leads to ambiguity or fatigue. As a results, we believe precise cursor-based interaction may be a valuable for AI interactions in XR, providing a high-fidelity mechanism to “ground” prompts to specific real-world targets without the ambiguity of gaze or the exertion of gesturing.

## 5\. Discussion

The World Mouse invites reconsideration of the dominant narrative that spatial computing must discard the desktop mouse in favor of novel or exotic interaction paradigms. Rather than positioning indirect input as obsolete, we believe our explorations suggest that precision and familiarity–hallmarks of the traditional mouse–still have great value when recontextualized for immersive and hybrid environments. By decoupling depth from physical reach, this approach establishes a low-fatigue interaction model that preserves dexterity while avoiding the inaccuracy and fatigue often associated with raycasting.

Moreover, our work illustrates that continuity can be a powerful strategy for designing cross-reality interactions. This interaction technique offers a bridge: it enables users to effortlessly flow between 2D and 3D spaces, to transition across digital and physical layers, and to interact with environments that are simultaneously virtual, real, and shared.

The implications extend beyond usability. As AI systems increasingly permeate our spatial environments by generating content, mediating interfaces, and controlling physical devices, there is a growing demand for tools that can express intent with both clarity and granularity. The World Mouse suggests one possible direction: a precision instrument for spatial design, AI guidance, and cross-device coordination. It can serve not just as an input device, but as a thinking tool in the age of spatial and generative computing.

While the World Mouse is designed to streamline 3D selection and confirmation tasks in XR, it is not intended for unconstrained freehand sketching [^7]. For scenarios requiring fluid, continuous 3D input (e.g., mid-air drawing or sculpting), traditional freehand methods may be better suited. Nevertheless, if XR becomes an essential interface to AI, much like screens have been for desktop computing, the proposed technique’s strength in precision and reduced fatigue could prove valuable. By allowing users to pinpoint where and how AI-driven tools (both large language models and computer vision systems) should focus, the World Mouse bridges physical and digital domains, supporting a range of emerging spatial workflows. We view this work not as a replacement for embodied interaction, but as a complementary trajectory – one that foregrounds precision, continuity, and long-term usability in spatial computing.

## 6\. Conclusion

In this work, we explored how the familiar form of a desktop mouse can be extended into immersive environments, offering a viable alternative for precise, low-fatigue interaction across varying depths and realities. The World Mouse bridges longstanding input metaphors with the demands of spatial computing, providing users with a consistent and intuitive control mechanism that remains effective even as environments shift between physical and digital. Rather than replacing gestural techniques, our approach contributes to a broader design landscape where diverse input modalities coexist and are tailored to specific user needs. As XR and AI systems continue to evolve, tools like the World Mouse may play a meaningful role in expanding interaction possibilities while preserving user agency.

[^2]: Karan Ahuja, Sujeath Pareddy, Robert Xiao, Mayank Goel, and Chris Harrison. 2019. Lightanchors: Appropriating point lights for spatially-anchored augmented reality interfaces. In *Proceedings of the 32nd Annual ACM Symposium on User Interface Software and Technology*. 189–196.

[^3]: Christoph Anthes, Rubén Jesús García-Hernández, Markus Wiedemann, and Dieter Kranzlmüller. 2016. State of the art of virtual reality technology. In *2016 IEEE aerospace conference*. IEEE, 1–19.

[^4]: Ferran Argelaguet and Carlos Andujar. 2013. A survey of 3D object selection techniques for virtual environments. *Computers & Graphics* 37, 3 (2013), 121–136.

[^5]: Iro Armeni, Zhi-Yang He, JunYoung Gwak, Amir R Zamir, Martin Fischer, Jitendra Malik, and Silvio Savarese. 2019. 3d scene graph: A structure for unified semantics, 3d space, and camera. In *Proceedings of the IEEE/CVF international conference on computer vision*. 5664–5673.

[^6]: Rahul Arora, Rubaiat Habib Kazi, Tovi Grossman, George Fitzmaurice, and Karan Singh. 2018. SymbiosisSketch: Combining 2D & 3D Sketching for Designing Detailed 3D Objects in Situ. In *Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems* (Montreal QC, Canada) *(CHI ’18)*. Association for Computing Machinery, New York, NY, USA, 1–15. [https://doi.org/10.1145/3173574.3173759](https://doi.org/10.1145/3173574.3173759)

[^7]: Rahul Arora and Karan Singh. 2021. Mid-Air Drawing of Curves on 3D Surfaces in Virtual Reality. *ACM Trans. Graph.* 40, 3, Article 33 (July 2021), 17 pages. [https://doi.org/10.1145/3459090](https://doi.org/10.1145/3459090)

[^8]: Richard A Bolt. 1980. “Put-that-there” Voice and gesture at the graphics interface. In *Proceedings of the 7th annual conference on Computer graphics and interactive techniques*. 262–270.

[^9]: Riccardo Bovo, Steven Abreu, Karan Ahuja, Eric J Gonzalez, Li-Te Cheng, and Mar Gonzalez-Franco. 2025. EmBARDiment: an Embodied AI Agent for Productivity in XR. In *2025 IEEE Conference Virtual Reality and 3D User Interfaces (VR)*. 708–717. [https://doi.org/10.1109/VR59515.2025.00093](https://doi.org/10.1109/VR59515.2025.00093)

[^10]: Andrew Bragdon, Rob DeLine, Ken Hinckley, and Meredith Ringel Morris. 2011. Code space: touch+ air gesture hybrid interactions for supporting developer meetings. In *Proceedings of the ACM International Conference on Interactive Tabletops and Surfaces*. 212–221.

[^11]: Yuan Chen, Géry Casiez, Sylvain Malacria, and Daniel Vogel. 2024. LuxAR: A Direct Manipulation Projected Display to Extend and Augment Desktop Computing. In *Proceedings of the 50th Graphics Interface Conference*. 1–12.

[^12]: Brian A Cohn, Antonella Maselli, Eyal Ofek, and Mar Gonzalez-Franco. 2020. Snapmove: Movement projection mapping in virtual reality. In *2020 IEEE international conference on artificial intelligence and virtual reality (AIVR)*. IEEE, 74–81.

[^13]: Mustafa Doga Dogan, Eric J Gonzalez, Karan Ahuja, Ruofei Du, Andrea Colaço, Johnny Lee, Mar Gonzalez-Franco, and David Kim. 2024. Augmented Object Intelligence with XR-Objects. In *Proceedings of the 37th Annual ACM Symposium on User Interface Software and Technology*. 1–15.

[^14]: Tomás Dorta, Gokce Kinayoglu, and Michael Hoffmann. 2016. Hyve-3D and the 3D Cursor: Architectural co-design with freedom in Virtual Reality. *International Journal of Architectural Computing* 14, 2 (2016), 87–102.

[^15]: Douglas C Engelbart and William K English. 1968. A research center for augmenting human intellect. In *Proceedings of the December 9-11, 1968, fall joint computer conference, part I*. 395–410.

[^16]: Andreas Rene Fender, Hrvoje Benko, and Andy Wilson. 2017. Meetalive: Room-scale omni-directional display system for multi-user content and control sharing. In *Proceedings of the 2017 ACM international conference on interactive surfaces and spaces*. 106–115.

[^17]: Clifton Forlines, Daniel Vogel, and Ravin Balakrishnan. 2006. Hybridpointing: fluid switching between absolute and relative pointing with a direct input device. In *Proceedings of the 19th annual ACM symposium on User interface software and technology*. 211–220.

[^18]: Scott Frees, G Drew Kessler, and Edwin Kay. 2007. PRISM interaction for enhancing control in immersive virtual environments. *ACM Transactions on Computer-Human Interaction (TOCHI)* 14, 1 (2007), 2–es.

[^19]: Eric J Gonzalez, Khushman Patel, Karan Ahuja, and Mar Gonzalez-Franco. 2024. XDTK: A Cross-Device Toolkit for Input & Interaction in XR. In *2024 IEEE Conference on Virtual Reality and 3D User Interfaces Abstracts and Workshops (VRW)*. IEEE, 467–470.

[^20]: Mar Gonzalez-Franco and Andrea Colaco. 2024. Guidelines for Productivity in Virtual Reality. *Interactions* 31, 3 (2024), 46–53.

[^21]: Mar Gonzalez-Franco, Zelia Egan, Matthew Peachey, Angus Antley, Tanmay Randhavane, Payod Panda, Yaying Zhang, Cheng Yao Wang, Derek F Reilly, Tabitha C Peck, et al. 2020. Movebox: Democratizing mocap for the microsoft rocketbox avatar library. In *2020 IEEE International Conference on Artificial Intelligence and Virtual Reality (AIVR)*. IEEE, 91–98.

[^22]: Mar Gonzalez-Franco, Mark Hall, Devon Hansen, Karl Jones, Paul Hannah, and Pablo Bermell-Garcia. 2015. Framework for remote collaborative interaction in virtual environments based on proximity. In *2015 IEEE Symposium on 3D User Interfaces (3DUI)*. IEEE, 153–154.

[^23]: Jens Emil Grønbæk, Mille Skovhus Knudsen, Kenton O’Hara, Peter Gall Krogh, Jo Vermeulen, and Marianne Graves Petersen. 2020. Proxemics beyond proximity: Designing for flexible social interaction through cross-device interaction. In *Proceedings of the 2020 CHI Conference on Human Factors in Computing Systems*. 1–14.

[^24]: Jens Emil Sloth Grønbæk, Ken Pfeuffer, Eduardo Velloso, Morten Astrup, Melanie Isabel Sønderkær Pedersen, Martin Kjær, Germán Leiva, and Hans Gellersen. 2023. Partially blended realities: Aligning dissimilar spaces for distributed mixed reality meetings. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems*. 1–16.

[^25]: Jeffrey T Hansberger, Chao Peng, Shannon L Mathis, Vaidyanath Areyur Shanthakumar, Sarah C Meacham, Lizhou Cao, and Victoria R Blakely. 2017. Dispelling the gorilla arm syndrome: the viability of prolonged gesture interactions. In *Virtual, Augmented and Mixed Reality: 9th International Conference, VAMR 2017, Held as Part of HCI International 2017, Vancouver, BC, Canada, July 9-14, 2017, Proceedings 9*. Springer, 505–520.

[^26]: Ken Hinckley. 2007. Input technologies and techniques. In *The human-computer interaction handbook*. CRC Press, 187–202.

[^27]: Ken Hinckley, Randy Pausch, Dennis Proffitt, and Neal F Kassell. 1998. Two-handed virtual manipulation. *ACM Transactions on Computer-Human Interaction (TOCHI)* 5, 3 (1998), 260–302.

[^28]: Ken Hinckley, Jeff Pierce, Mike Sinclair, and Eric Horvitz. 2000. Sensing techniques for mobile interaction. In *Proceedings of the 13th annual ACM symposium on User interface software and technology*. 91–100.

[^29]: Christian Holz and Patrick Baudisch. 2011. Understanding touch. In *Proceedings of the SIGCHI Conference on Human Factors in Computing Systems*. 2501–2510.

[^30]: Daekun Kim and Daniel Vogel. 2022. Everywhere Cursor: Extending Desktop Mouse Interaction into Spatial Augmented Reality. In *CHI Conference on Human Factors in Computing Systems Extended Abstracts*. 1–7.

[^31]: Jinwook Kim, Sangmin Park, Qiushi Zhou, Mar Gonzalez-Franco, Jeongmi Lee, and Ken Pfeuffer. 2025. PinchCatcher: Enabling Multi-selection for Gaze+Pinch. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems* *(CHI ’25)*. Association for Computing Machinery, New York, NY, USA, Article 853, 16 pages. [https://doi.org/10.1145/3706598.3713530](https://doi.org/10.1145/3706598.3713530)

[^32]: Alexander Kirillov, Eric Mintun, Nikhila Ravi, Hanzi Mao, Chloe Rolland, Laura Gustafson, Tete Xiao, Spencer Whitehead, Alexander C Berg, Wan-Yen Lo, et al. 2023. Segment anything. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 4015–4026.

[^33]: Alexandra Kitson, Sun Joo Ahn, Eric J Gonzalez, Payod Panda, Katherine Isbister, and Mar Gonzalez-Franco. 2024. Virtual Games, Real Interactions: A Look at Cross-reality Asymmetrical Co-located Social Games. In *Extended Abstracts of the CHI Conference on Human Factors in Computing Systems*. 1–9.

[^34]: Werner A König, Jens Gerken, Stefan Dierdorf, and Harald Reiterer. 2009. Adaptive pointing–design and evaluation of a precision enhancing technique for absolute pointing devices. In *Human-Computer Interaction–INTERACT 2009: 12th IFIP TC 13 International Conference, Uppsala, Sweden, August 24-28, 2009, Proceedings, Part I 12*. Springer, 658–671.

[^35]: Mikko Kytö, Barrett Ens, Thammathip Piumsomboon, Gun A Lee, and Mark Billinghurst. 2018. Pinpointing: Precise head-and eye-based target selection for augmented reality. In *Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems*. 1–14.

[^36]: Sangyoon Lee, Jinseok Seo, Gerard Jounghyun Kim, and Chan-Mo Park. 2003. Evaluation of pointing techniques for ray casting selection in virtual environments. In *Third international conference on virtual reality and its application in industry*, Vol. 4756. SPIE, 38–44.

[^37]: Mathias N. Lystbæk, Thorbjørn Mikkelsen, Roland Krisztandl, Eric J Gonzalez, Mar Gonzalez-Franco, Hans Gellersen, and Ken Pfeuffer. 2024. Hands-on, Hands-off: Gaze-Assisted Bimanual 3D Interaction. In *Proceedings of the 37th Annual ACM Symposium on User Interface Software and Technology* (Pittsburgh, PA, USA) *(UIST ’24)*. Association for Computing Machinery, New York, NY, USA, Article 80, 12 pages. [https://doi.org/10.1145/3654777.3676331](https://doi.org/10.1145/3654777.3676331)

[^38]: Hongyu Mao, Mar Gonzalez-Franco, Vrushank Phadnis, Eric J Gonzalez, and Ishan Chatterjee. 2025. RestfulRaycast: Exploring Ergonomic Rigging and Joint Amplification for Precise Hand Ray Selection in XR. In *Proceedings of the 2025 ACM Designing Interactive Systems Conference* *(DIS ’25)*. Association for Computing Machinery, New York, NY, USA, 28–39. [https://doi.org/10.1145/3715336.3735677](https://doi.org/10.1145/3715336.3735677)

[^39]: Nicolai Marquardt, Ken Hinckley, and Saul Greenberg. 2012. Cross-device interaction via micro-mobility and f-formations. In *Proceedings of the 25th annual ACM symposium on User interface software and technology*. 13–22.

[^40]: Thorbjørn Mikkelsen, Qiushi Zhou, Mar Gonzalez-Franco, Hans Gellersen, and Ken Pfeuffer. 2026. Preshaping Hand Behaviour for Direct and Indirect Manipulation of 3D Objects. In *Proceedings of the 2026 CHI Conference on Human Factors in Computing Systems, CHI*, Vol. 26.

[^41]: Roberto A Montano Murillo, Sriram Subramanian, and Diego Martinez Plasencia. 2017. Erg-O: Ergonomic optimization of immersive virtual environments. In *Proceedings of the 30th annual ACM symposium on user interface software and technology*. 759–771.

[^42]: Eduardo GQ Palmeira, Alexandre Campos, Ígor A Moraes, Alexandre G de Siqueira, and Marcelo GG Ferreira. 2023. Quantifying the’Gorilla Arm’Effect in a Virtual Reality Text Entry Task via Ray-Casting: A Preliminary Single-Subject Study. In *Proceedings of the 25th Symposium on Virtual and Augmented Reality*. 274–278.

[^43]: Payod Panda, Molly Jane Nicholas, David Nguyen, Eyal Ofek, Michel Pahud, Sean Rintel, Mar Gonzalez-Franco, Ken Hinckley, and Jaron Lanier. 2023. Beyond audio: Towards a design space of headphones as a site for interaction and sensing. In *Proceedings of the 2023 ACM Designing Interactive Systems Conference*. 904–916.

[^44]: Siyou Pei, David Kim, Alex Olwal, Yang Zhang, and Ruofei Du. 2024. UI Mobility Control in XR: Switching UI Positionings between Static, Dynamic, and Self Entities. In *Proceedings of the CHI Conference on Human Factors in Computing Systems*. 1–12.

[^45]: Ken Pfeuffer, Hans Gellersen, and Mar Gonzalez-Franco. 2024. Design principles and challenges for gaze+ pinch interaction in xr. *IEEE Computer Graphics and Applications* 44, 3 (2024), 74–81.

[^46]: Ken Pfeuffer, Benedikt Mayer, Diako Mardanbegi, and Hans Gellersen. 2017. Gaze+ pinch interaction in virtual reality. In *Proceedings of the 5th symposium on spatial user interaction*. 99–108.

[^47]: Ivan Poupyrev, Mark Billinghurst, Suzanne Weghorst, and Tadao Ichikawa. 1996. The go-go interaction technique: non-linear mapping for direct manipulation in VR. In *Proceedings of the 9th annual ACM symposium on User interface software and technology*. 79–80.

[^48]: Junwei Sun, Wolfgang Stuerzlinger, and Bernhard E Riecke. 2018a. Comparing input methods and cursors for 3D positioning with head-mounted displays. In *Proceedings of the 15th ACM Symposium on Applied Perception*. 1–8.

[^49]: Junwei Sun, Wolfgang Stuerzlinger, and Bernhard E. Riecke. 2018b. Comparing input methods and cursors for 3D positioning with head-mounted displays. In *Proceedings of the 15th ACM Symposium on Applied Perception* (Vancouver, British Columbia, Canada) *(SAP ’18)*. Association for Computing Machinery, New York, NY, USA, Article 8, 8 pages. [https://doi.org/10.1145/3225153.3225167](https://doi.org/10.1145/3225153.3225167)

[^50]: Junjiao Tian, Lavisha Aggarwal, Andrea Colaco, Zsolt Kira, and Mar Gonzalez-Franco. 2024. Diffuse Attend and Segment: Unsupervised Zero-Shot Segmentation using Stable Diffusion. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 3554–3563.

[^51]: Esen K. Tütüncü, Mar Gonzalez-Franco, and Eric J Gonzalez. 2025. HandOver: Enabling Precise Selection & Manipulation of 3D Objects with Mouse and Hand Tracking. In *Proceedings of the 38th Annual ACM Symposium on User Interface Software and Technology* *(UIST ’25)*. Association for Computing Machinery, New York, NY, USA, Article 97, 11 pages. [https://doi.org/10.1145/3746059.3747757](https://doi.org/10.1145/3746059.3747757)

[^52]: Cheng Yao Wang, Eyal Ofek, Hyunju Kim, Payod Panda, Andrea Stevenson Won, and Mar Gonzalez Franco. 2024. AvatarPilot: Decoupling one-to-one motions from their semantics with weighted interpolations. In *2024 IEEE International Symposium on Mixed and Augmented Reality Adjunct (ISMAR-Adjunct)*. IEEE Computer Society, 588–591.

[^53]: Johann Wentzel, Matthew Lakier, Jeremy Hartmann, Falah Shazib, Géry Casiez, and Daniel Vogel. 2024. A Comparison of Virtual Reality Menu Archetypes: Raycasting, Direct Input, and Marking Menus. *IEEE Transactions on Visualization and Computer Graphics* (2024).

[^54]: Haijun Xia, Tovi Grossman, and George Fitzmaurice. 2015. NanoStylus: Enhancing input on ultra-small displays with a finger-mounted stylus. In *Proceedings of the 28th Annual ACM Symposium on User Interface Software & Technology*. 447–456.

[^55]: Haijun Xia, Ken Hinckley, Michel Pahud, Xiao Tu, and Bill Buxton. 2017. WritLarge: Ink Unleashed by Unified Scope, Action, & Zoom.. In *CHI*, Vol. 17. 3227–3240.

[^56]: Chenyang Zhang, Tiffany S Ma, John Andrews, Eric J Gonzalez, Mar Gonzalez-Franco, and Yalong Yang. 2025. ForcePinch: Force-Responsive Spatial Interaction for Tracking Speed Control in XR. In *Proceedings of the 38th Annual ACM Symposium on User Interface Software and Technology* *(UIST ’25)*. Association for Computing Machinery, New York, NY, USA, Article 27, 16 pages. [https://doi.org/10.1145/3746059.3747694](https://doi.org/10.1145/3746059.3747694)

[^57]: Qian Zhou, George Fitzmaurice, and Fraser Anderson. 2022. In-depth mouse: Integrating desktop mouse into virtual reality. In *Proceedings of the 2022 CHI Conference on Human Factors in Computing Systems*. 1–17.

[^58]: Yunzhan Zhou, Lei Shi, Zexi He, Zhaoxing Li, and Jindi Wang. 2023. Design paradigms of 3D user interfaces for VR exhibitions. In *IFIP Conference on Human-Computer Interaction*. Springer, 618–627.

[^59]: Fengyuan Zhu and Tovi Grossman. 2020. Bishare: Exploring bidirectional interactions between smartphones and head-mounted augmented reality. In *Proceedings of the 2020 CHI Conference on Human Factors in Computing Systems*. 1–14.

[^60]: Fengyuan Zhu, Mauricio Sousa, Ludwig Sidenmark, and Tovi Grossman. 2024. PhoneInVR: An Evaluation of Spatial Anchoring and Interaction Techniques for Smartphone Usage in Virtual Reality. In *Proceedings of the CHI Conference on Human Factors in Computing Systems*. 1–16.