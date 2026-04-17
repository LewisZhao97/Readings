---
title: "Vibe Coding XR: Accelerating AI + XR Prototyping with XR Blocks and Gemini"
author:
  - "Ruofei Du"
  - "Benjamin Hersh"
source: "https://arxiv.org/html/2603.24591v2"
created: 2026-04-17
tags:
  - xr
  - ai-xr
  - vibe-coding
  - xr-blocks
  - webxr
  - llm-code-generation
  - benchmark
  - prototyping
status: true
ingested: 2026-04-17
---
Ruofei Du <sup>∗‡</sup>, Benjamin Hersh, David Li <sup>∗</sup>, Nels Numan <sup>†</sup>, Xun Qian <sup>†</sup>, Yanhe Chen <sup>†</sup>, Zhongyi Zhou, Jiahao Ren, Xingyue Chen, Robert Timothy Bettridge, Faraz Faruqi, Xiang ‘Anthony’ Chen and Steve Toh, David Kim [https://xrblocks.github.io/gem](https://xrblocks.github.io/gem)  
[https://github.com/google/xrblocks](https://github.com/google/xrblocks)  
Google XR Labs

###### Abstract.

While large language models (LLMs) have accelerated 2D software development through intent-driven “vibe coding”, prototyping intelligent Extended Reality (XR) experiences remains a major challenge. The fundamental barrier is not just the steep learning curve for human creators, but that low-level sensor APIs and complex game engine hierarchies are ill-suited for LLM reasoning, routinely exceeding context windows and inducing syntax hallucinations. To bridge this gap, we contribute XR Blocks, an open-source, LLM-native WebXR framework. Unlike traditional engines, XR Blocks introduces a semantic “Reality Model” that aligns spatial computing primitives (users, physical environments, and agents) with natural language, providing a robust, concise vocabulary optimized for generative AI. Building upon this foundation, we present Vibe Coding XR, an end-to-end prototyping workflow that leverages LLMs to translate high-level prompts (e.g., “create a dandelion that reacts to my hand”) directly into functional, physics-aware mixed-reality applications. To minimize the friction of on-device testing, the workflow introduces a seamless desktop “simulated reality” to headset deployment loop. Finally, we introduce VCXR60, a pilot dataset of 60 XR prompts paired with an automated evaluation pipeline. Our technical evaluation demonstrates high one-shot execution success, enabling practitioners to bypass low-level hurdles and rapidly move from “idea to reality.” Code and live demos are available at [https://github.com/google/xrblocks](https://github.com/google/xrblocks) and [http://xrblocks.github.io/gem](http://xrblocks.github.io/gem).

vibe coding, extended reality, XR Blocks, prototyping, AI, XR, GenXR

<sup>∗</sup> Both authors contribute equally to XR Blocks.  
<sup>†</sup> Equal contributions, sorted alphabetically.  
<sup>‡</sup> Corresponding author: Ruofei Du, me \[at\] duruofei \[dot\] com.

![Refer to caption](https://arxiv.org/html/2603.24591v2/x1.png)

Figure 1. Example user journey of Vibe Coding XR, an end-to-end workflow for creating immersive AI + XR experiences via vibe coding: (A) User types “create a beautiful dandelion” with XR Blocks Gem ( http://xrblocks.github.io/gem ) on a Galaxy XR headset in a Chrome browser. (B) Gemini translates the input into an interactive XR application within a minute, while the user reviews its reasoning and coding process. (C) User selects the “Enter XR” button and sees an animated dandelion dispersing upon pinch.

## 1\. Introduction

![Refer to caption](https://arxiv.org/html/2603.24591v2/x2.png)

Figure 2. Vibe Coding XR accelerates AI + XR prototyping by allowing users to (A) test their “vibe coding” results on desktop in a “simulated reality” environment, and (B) deploy the same demo on an Android XR headset with body and hand interactions.

Recent advances in Large Language Models (LLMs) [^49] [^10] [^44] [^29] and agentic workflows [^25] [^28] [^13] are fundamentally reshaping software engineering and creative computing. We are witnessing the growing prevalence of “vibe coding” [^20], a paradigm in which high-level human intent is translated directly into functional software. While tools such as Gemini Canvas [^26], Antigravity [^28], Cursor [^13], and Claude Code [^5] have successfully expanded this capability for 2D and, more recently, 3D web development [^50] [^19], the domain of Extended Reality (XR) remains largely inaccessible. Prototyping intelligent spatial experiences still requires navigating a fragmented ecosystem of perception pipelines [^54] [^42] [^30], low-level sensor APIs [^37] [^21], massive game engine hierarchies [^46] [^48], and the physical friction of constant on-device testing.

Crucially, the absence of “vibe coding” in XR stems from a fundamental mismatch between how spatial software is currently built and how LLMs reason. Current models struggle to generate reliable XR applications because they are forced to operate over massive, low-level syntax trees and fragmented API layers rather than semantic, human-centered spatial concepts. This incidental complexity routinely exhausts context windows, fragments reasoning steps, and induces code hallucinations. To enable reliable intent-driven spatial computing, we need an abstraction layer explicitly designed not just for human accessibility, but for machine reasoning.

To bridge this gap, we introduce XR Blocks [^1] [^36], an open-source WebXR [^7] framework designed specifically to make spatial computing intelligible to generative AI. As the foundational contribution of this work, XR Blocks introduces an LLM-native “Reality Model” that treats the user, the physical environment, and intelligent agents as first-class, configurable primitives. By encapsulating the complexities of sensor fusion, rendering, and spatial physics into a concise, semantic vocabulary, XR Blocks provides LLMs with the precise architectural constraints necessary to generate complex spatial behaviors without hallucinating underlying implementation.

Building upon this enabling framework, we present Vibe Coding XR, an end-to-end rapid prototyping workflow that leverages the long-context reasoning of LLMs to act as an expert XR designer. Using a specialized system prompt and a web-based interface <sup>2</sup>, this workflow translates natural language directly into deployable, physics-aware XR applications in under 90 seconds. To solve the persistent HCI challenge of spatial testing friction, Vibe Coding XR allows creators to rapidly iterate on generated scripts within a desktop “simulated reality” environment before deploying instantly to an Android XR [^24] headset for live hand interaction and environmental sensing.

Finally, evaluating the viability of generated XR software requires moving beyond subjective manual testing. We contribute the VCXR60 dataset, a pilot dataset consisting of 60 diverse spatial computing prompts sourced from four workshops. Coupled with an automated, headless-browser evaluation pipeline, we provide a quantitative baseline for GenXR code generation. Our evaluation, alongside diverse application scenarios demonstrating XR realism and multi-modal interaction, illustrates the robustness of the system. In summary, we contribute:

- The XR Blocks Framework: An open-source, LLM-native WebXR architecture that abstracts low-level spatial perception and interaction pipelines into a semantic “Reality Model” optimized for reliable AI code generation.
- Vibe Coding XR Workflow: A web-based, rapid prototyping loop that leverages LLMs to translate natural language intent into deployable XR applications, utilizing a novel desktop-simulator-to-headset testing pipeline to reduce iterative friction.
- The VCXR60 Dataset & Evaluation: A preliminary dataset and automated testing pipeline for evaluating intent-driven XR code generation, demonstrating the high one-shot viability of our framework across varied applications.

## 2\. Related Work

Vibe Coding XR bridges the gap between generative AI workflows and spatial computing by enabling intent-driven XR prototyping.

![Refer to caption](https://arxiv.org/html/2603.24591v2/x3.png)

Figure 3. Design of the XR Blocks Framework: (A) The “Reality Model” conceptual abstraction, which aligns spatial computing primitives 1:1 with natural language concepts to prevent LLM hallucination over fragmented syntax trees. (B) The modular architecture of the “core” engine encapsulating low-level perception and interaction logic. Subsystems marked with ∗ have not yet been fully open sourced.

### 2.1. GenXR: Creating AI + XR Experiences

XR development tooling has evolved from early toolkits like VR Juggler [^8] and ARToolKit [^34] to comprehensive game engines such as Unity [^46] and Unreal [^48]. While these native engines offer high ceilings for fidelity, they impose steep learning curves, significant friction for rapid prototyping for XR, and high barriers for sharing executable code. Conversely, WebGL libraries like three.js [^45] and A-Frame [^4] provide cross-platform accessibility, but lack the high-level abstractions necessary for complex XR interaction and seamless AI integration. XR Blocks bridges this divide. By offering a high-level, web-optimized interaction model akin to MRTK [^6], VRTK [^51], XRI [^47] for Unity, our framework abstracts low-level sensor and perception pipelines so creators can focus on the what of an experience rather than the how. As a web-native architecture, XR Blocks makes AI + XR prototyping inherently accessible and reproducible, enabling the community to share, fork, and build upon each other’s creations. Vibe Coding XR further leverages this framework to create behavioral AI + XR experiences, with 3D assets creation [^41] [^43] [^52] [^32] remaining out of scope.

### 2.2. Vibe Coding: Gaps from AI to XR

The AI community benefits from a “flywheel effect” driven by open ecosystems such as Hugging Face [^33] and TensorFlow Hub [^27], benchmarks like LMArena [^12], and composable frameworks including JAX [^9], PyTorch [^39], and TensorFlow [^3]. XR, however, lacks a comparable substrate for rapid, intent-driven iteration. Recent efforts have begun to address this by leveraging LLMs to generate spatial objects and scenes in Unity, such as LLMR [^14], DreamCodeVR [^22], and Thing2Reality [^32], with commercial tools such as Bezi [^2] pursuing a similar direction. On the infrastructure side, Ubiq-Genie [^38] proposed a server-client architecture that exposes AI services to Unity-based XR applications. Furthermore, recent systems like DreamGarden [^19] have demonstrated the use of LLM-driven hierarchical planning to generate functional 3D game environments, assets, and C++ logic in Unreal Engine.

While these approaches successfully push the boundaries of automated game design and 3D scene composition, they often rely on asynchronous compilation workflows within heavy, closed-engine ecosystems. This limits the collective “vibe coding” paradigm [^20], in which an agent’s utility is fundamentally tied to its ability to synthesize and share knowledge across the entire relevant domain. To this end, we develop a spatial computing platform on the open web inspired by [^16]. By coupling LLMs [^53] directly with the XR Blocks Reality Model, Vibe Coding XR enables creators to generate immersive, intelligent, and physics-aware interactions. This web-based approach ensures that every contribution is (1) built upon a transparent, web-standard codebase, (2) designed for rapid iteration, and (3) instantly distributable across diverse devices and users. By prioritizing these pillars, we move toward an ecosystem where the community does not merely consume agents but actively drives their evolution, supporting a “flywheel effect” for XR.

## 3\. System Architecture

We introduce Vibe Coding XR, a rapid prototyping workflow that combines the generative reasoning capabilities of LLMs with a high-level modular XR framework. This combination allows creators to bypass the friction of low-level systems programming and focus on designing spatial behaviors, AI logic, and interactions.

### 3.1. The XR Blocks Reality Model

At the foundation of our system is XR Blocks, an open-source LLM-native abstraction layer designed to make spatial computing intelligible to generative AI. Traditional game engines and low-level WebGL APIs represent scenes as massive, deeply nested syntax trees and fragmented data streams. When forced to navigate these structures, LLMs routinely exhaust their context windows, lose track of spatial relationships, and hallucinate API calls. To solve this, XR Blocks introduces a semantic Reality Model. This unified abstraction aligns spatial computing primitives (the user, the physical environment, and intelligent agents) directly with natural language concepts, providing a robust, concise vocabulary optimized for machine reasoning. The following snippet demonstrates XR Blocks’s design principle of keeping things simple and semantically dense for the LLM: it creates an object at the user’s eye level with a preferred distance to the object, which changes color when the user performs a hand pinch or desktop click.

![Refer to caption](https://arxiv.org/html/2603.24591v2/x4.png)

Figure 4. Human-coded templates and samples in the XR Blocks framework provide the foundational best practices and API grounding for Vibe Coding XR.

![[Uncaptioned image]](https://arxiv.org/html/2603.24591v2/x5.png)

\[Uncaptioned image\]

As shown in Figure 3, the framework’s core engine manages the complex interplay of subsystems required for spatial computing. This includes environmental perception (e.g., depth sensing, lighting estimation), XR interaction (e.g., hand rays, pinch gestures, rigid-body physics), and AI integration (e.g., LLM querying, agent memory). By abstracting these technical hurdles, XR Blocks lowers the barrier to entry while retaining a high ceiling for expressivity, positioning it as the ideal foundation for Vibe Coding XR.

The framework’s architecture was iteratively refined through the development of 34 open-source templates (examples: Figure 4), incorporating feedback from weekly workshops and live demos.

### 3.2. System Prompt Architecture

Vibe Coding XR leverages the long-context capabilities and advanced thinking processes of modern LLMs like Gemini [^44] to function as an expert XR designer and engineer. Merely providing an API is insufficient; the LLM must be constrained and guided. We developed a specialized system prompt architecture in Appendix C that explicitly “teaches” the LLM the XR Blocks Reality Model through an open-source web interface. This includes:

- Persona & Guidelines: Configures the LLM with a domain expert persona adhering to best practices for room-scale XR environments (e.g., spatial layout, human-scale proportions, and interaction distances).
- Package Management: Specifies how dependencies within XR Blocks are handled to enforce recommended styles.
- Source Code & Templates: Grounds the model with the full source code of the core xrblocks.js library, alongside curated templates and samples (illustrated in Figure 4). This strict grounding acts as a technical constraint, minimizing API hallucination and enforcing adherence to established spatial design patterns.

### 3.3. The Simulated-to-Extended Reality Loop

Users interact with Vibe Coding XR through high-level natural language prompts (or “vibes”). The LLM translates this intent directly into functional XR Blocks scripts [^53]. Crucially, to solve the persistent HCI friction of iterative spatial testing, where creators must constantly don and doff a headset, the workflow introduces a seamless simulated to extended reality loop. Scripts can be previewed immediately in a desktop simulated reality environment, allowing for rapid visual validation of physics, logic, and layout before being deployed directly to an Android XR headset for real-world interaction testing.

### 3.4. Application Scenarios: From Prompt to XR

To demonstrate the versatility of the Vibe Coding XR workflow, we present several prototypes generated via Vibe Coding XR:

![Refer to caption](https://arxiv.org/html/2603.24591v2/x6.png)

Figure 5. AI-generated application for educational and exercise use cases. See Appendix B for full prompts.

#### Educational AI + XR Experiences

As shown in Figure 5, Vibe Coding XR supports rapid prototyping of interactive learning environments. Prompted with “visualize Euler’s theorem in geometry and explain vertices, edges, and facets” (AS1), it generated an immersive Math Tutor application. The LLM autonomously selected geometries (a tetrahedron, cube, and octahedron) and allowed users to pinch them to trigger different highlighting strategies. Similarly, prompting for an Immersive Chemistry (AS3) application produced an experience featuring educational cards and 3D volumetric visual effects, allowing users to safely observe simulated reactions (e.g., igniting methane or ethylene) via hand-pinch gestures.

#### Physics-Aware XR Realism & Interaction

By grounding the LLM in XR Blocks’s physics and depth modules, users can generate highly tactile experiences. Prompting for an XR Sports (AS4) application (“Let me play volleyball with hands and collide with my environment”) yielded a textured ball that reacts realistically to both the user’s physical hands and the surrounding room geometry. Furthermore, a Physics Lab prompt (AS2) successfully implemented a mixed-reality scale where users pick and drop labeled weights to intuitively learn mechanics.

#### Game Prototyping & Procedural Generation

This workflow substantially reduces the time required to prototype XR games. Prompted to “create the Chrome Dino game in XR,” (AS5) the system generated a voxelized version of the classic game, complete with rushing cacti on a semi-transparent lane and audio cues, reducing development time from hours to minutes.

## 4\. Preliminary Evaluation with VCXR60

Evaluating XR applications poses unique challenges, as it typically requires manual, on-device testing and subjective human evaluation. However, assessing the viability of generative XR systems requires fundamentally new, scalable technical metrics that can keep pace with the rapid iteration of large language models. To meet this need, we introduce the VCXR60 dataset, an open-source benchmark designed to automate the testing of intent-driven spatial computing.

Sourced from four one-hour workshops, VCXR60 consists of 60 XR application prompts provided by 20 participants (detailed in Appendix A). Using this dataset with XR Blocks’s desktop simulator, we measured inference time and one-shot pass rates (pass@1). Inspired by code LLM benchmarks like HumanEval [^11], we define a “pass” as a zero-error execution in an automated runtime test. In 3D graphics, the rendering engine itself serves as an implicit correctness validator, as runtime errors during rendering indicate code failures. We therefore monitor error logs in a headless Chromium browser via Playwright [^40] as our automated test harness.

Early analysis revealed that the majority of generation errors stemmed from edge-case bugs within the XR Blocks framework itself (v0.1.0) or hallucinated APIs by the LLMs, initially yielding an approximate 70% pass rate. These insights informed a rapid six-month iteration cycle. After 10 major releases, we evaluated XR Blocks v0.11.0 to gather the baseline results shown in Table 1.

Averaged across five runs, our evaluation revealed distinct performance trade-offs based on model size, “thinking” levels, and prompt complexity. Notably, every test prompt achieved at least one successful execution across the five runs. For simpler interactions (e.g., “Create a beautiful dandelion that blows away when I pick it up”), faster models like Gemini Flash often completed generation in under 20 seconds. For applications requiring complex animations and precise hand-interaction logic (e.g., Prompt 060 in Appendix A), however, these models exhibited a higher rate of runtime errors compared to more advanced models like Gemini Pro.

Table 1. Inference time and one-shot success rate for XR Blocks Gem with various Gemini backends on VCXR60 in 5 runs. We used “preview” models in the evaluation.

<table><thead><tr><th colspan="2">XR Blocks Gem</th><th colspan="2">Time (secs)</th><th>Pass Rate@1</th></tr><tr><th>Gemini Model</th><th>Thinking</th><th>Median</th><th>IQR</th><th>Percentage</th></tr></thead><tbody><tr><td>gemini-3.1-pro</td><td>High</td><td>86.02</td><td>32.12</td><td>95.5%</td></tr><tr><td>gemini-3.1-pro</td><td>Low</td><td>33.39</td><td>8.60</td><td>94.1%</td></tr><tr><td>gemini-3-flash</td><td>High</td><td>22.26</td><td>4.66</td><td>87.8%</td></tr><tr><td>gemini-3-flash</td><td>Low</td><td>17.30</td><td>4.00</td><td>87.4%</td></tr></tbody></table>

## 5\. Limitations and Discussion

This paper presents our initial step towards “idea to reality”. Our current implementation prioritizes the core abstractions for XR, and its limitations highlight exciting avenues for future research.

#### Reliability of LLM-Generated Code

The stochastic nature of LLMs introduces friction into the prototyping loop. Generated applications occasionally misinterpret physical constraints, produce invalid syntax, or fail to implement complex interaction logic. Our evaluation shows that larger models with extended reasoning budgets mitigate these issues considerably, but do not eliminate them. A systematic investigation of failure modes, particularly for prompts that require tightly coupled perception and interaction logic, would help identify where additional framework-level abstractions or constrained generation strategies could improve reliability.

#### Performance and Platform Trade-offs

Our choice of web technologies prioritizes accessibility and shareability but introduces known trade-offs. The framework cannot match the rendering throughput of native engines like Unity or Unreal, and reliance on cloud-hosted LLMs adds network latency to the generation step. A longer-term goal is to develop an LLM-driven cross-compiler capable of translating high-level XR Blocks scripts into optimized, native code for target engines such as Godot [^23]. We invite the community to contribute to the SDK to establish shared conventions for AI + XR interactions and to further reduce LLM token costs.

#### Toward Rigorous Benchmarking

VCXR60 covers a limited portion of the AI + XR design space, and our automated pass/fail metric captures code viability but does not assess usability, aesthetic quality, or interaction fidelity. A rigorous evaluation should decouple the contributions of XR Blocks from those of the underlying three.js library and quantify how many tokens the framework’s abstractions save relative to raw API usage. Evaluation should also address iterative, multi-turn usage scenarios, such as refining an existing prototype in response to evolving requirements, which may expose distinct failure modes in context management and incremental code coherence. Just as ImageNet [^15] accelerated progress in computer vision, the XR and HCI fields would benefit from large-scale, open-source datasets of prompt-to-interaction pairs that enable systematic evaluation of generated spatial behaviors through human raters, expert review, and agentic testing workflows. We aim to evolve XR Blocks alongside advances in LLMs through continued open-source collaboration with communities. For example, XR Blocks currently lacks ready-to-use accessibility and safety modules, an important gap given the framework’s goal of broad participation. Future iterations of Vibe Coding XR would benefit from rigorous human-in-the-loop evaluation to understand how designers co-create with AI agents.

## 6\. Conclusion

We present Vibe Coding XR, an end-to-end workflow that democratizes spatial computing by translating high-level creative intent into functional XR prototypes. By coupling the generative reasoning of LLMs with the XR Blocks framework, we demonstrated a novel approach that collapses the distance between a fleeting thought and a tangible, physics-aware reality. This shift toward spatial “vibe coding” empowers a new generation of creators to shape the 3D web, transforming passive consumers into active architects of their digital and physical environments.

However, this human-AI symbiosis remains in its infancy. The stochastic nature of current LLMs still introduces friction, occasionally generating code that misinterprets physical constraints, syntax, or complex interaction logic. Furthermore, XR Blocks currently lacks ready-to-use accessibility modules. To mature this domain, the field must move beyond ad-hoc demonstrations toward rigorous, formal benchmarking. Just as ImageNet [^15] accelerated progress in computer vision, we envision large-scale, open-source datasets of prompt-to-interaction pairs. Such benchmarks will enable the systematic evaluation of generated spatial behaviors through human raters, expert review, and agentic workflows. Hybrid approaches that blend natural language prompting with spatial UIs will also be critical for refining XR creation.

As an evolving ecosystem, future iterations of XR Blocks must expand its generative vocabulary to support improved aesthetics and richer multimodal inputs. This will enable users to guide generation not only through text and voice, but also via gaze, micro-gestures, and cross-device interactions. Concurrently, responsible real-world deployment necessitates rigorous human-in-the-loop evaluation to better understand how designers co-create with AI agents, ensuring these tools safely augment human intent rather than introduce harmful noise.

Ultimately, we release this framework, workflow, and preliminary dataset as an open invitation. We welcome the HCI, XR, and AI communities to build upon this foundation, expand the capabilities of generative spatial computing, and collectively work toward a future where moving from “idea to reality” is as fluid as describing it, and eventually kicking off the flywheel of accelerating AI + XR innovations together.

###### Acknowledgements.

We thank all of XR Blocks contributers in 2025: David Li and Ruofei Du (equal primary contributions), Nels Numan, Xun Qian, Yanhe Chen, and Zhongyi Zhou, (equal secondary contributions, sorted alphabetically), as well as Evgenii Alekseev, Geonsun Lee, Alex Cooper, Brandon Jones, Min Xia, Scott Chung, Jeremy Nelson, Xiuxiu Yuan, Jolica Dias, Tim Bettridge, Benjamin Hersh, Michelle Huynh, Konrad Piascik, Ricardo Cabello, and David Kim. We further thank the Gemini Canvas and AI Studio teams for their support including, but not limited to: Tim Bettridge, Yan Li, Daniel Marques, Deven Tokuno, Levent Yilmaz, Saravana Rathinam, Samuel Petit, Mike Taylor-Cai, Ammaar Reshi, and Robert Berry, We would like to thank Mahdi Tayarani, Max Dzitsiuk, Jim Ratcliffe, Patrick Hackett, Seeyam Qiu, Coco Fatus, Alon Hetzroni, Aaron Kim, Yinghua Yang, Brian Collins, Eric Gonzalez, Keith Moon, Nicolás Peña Moreno, Yidang Zhang, Jamie Pepper, Yuhao He, Yi-Fei Li, Ziyi Liu, Jing Jin for their feedback and discussion on our early-stage proposal and WebXR experiments, as well as all of DepthLab [^18], Ad hoc UI [^17], Rapsai (Visual Blocks) [^16], InstructPipe [^53], DialogueLab [^31], and Sensible Agent [^35] co-authors, which greatly inspired this project along the way. We appreciate Tim Herrmann and Andrew Helton’s thoughtful reviews. We thank Maryam Sanglaji, Max Spear, Adarsh Kowdle, and Guru Somadder, Shahram Izadi for the directional feedback and contribution.

## References

## Appendix A VCXR60 Dataset Prompts

001: blowing\_dandelion:

Create a beautiful dandelion. when I pick it up and hold with my hands, make the seeds blow away.

002: rainbow\_pen:

Make a pen that draws rainbows in 3D.

003: vine\_growth:

Create a vine growth animation with tendrill curling, leaf, sprouting, wall climbing, flower blooming, and organic spreading pattern.

004: bubble\_pop:

Make a bunch of bubbles that pop when I touch them.

005: origami\_cherry\_blossoms:

Origami cherry blossoms that fall when you pick the first petal:cherry-blossom-cowboy: Make pedals collide with physical environment using depth mesh.

006: wall\_aquarium:

Create a new app, When I click a detected mesh of the wall, a virtual fish tank should be created and carved into that detective mesh. The final result should looks like there is a fish tank which inside the wall.

007: cat\_platformer:

Create a cat-themed super mario alike game.

008: planetary\_gearbox:

Create a model of a planetary gearbox and display it in the app. Animate the model to demonstrate how the device works. Add teeth to the gears in the model to make the rotation of the gears easier to see.

009: guitar\_tab\_tutor:

Create an app with a model of a guitar that shows dots on strings and frets that correlate to tablature and show you how to play a song. Use Jimi Hendrix little wing as an example.

010: hand\_fire:

Create a WebXR application called ”Hand Fire” using three.js and xrblocks that tracks both hands to emit distinct particle-based fire effects—blue fire for the left hand and orange fire for the right. Implement a custom CPU-driven particle system with a shader for the visual effect, and include gesture logic where pinching makes the fire follow the fingertips, while a ”thumbs up” gesture freezes the fire in mid-air at its current location. Additionally, provide a spatial UI panel with an ”Ignite Hands” toggle button that forces the fire to appear immediately (floating in front of the camera if hands are not detected, or snapping to the palm center if they are), allowing for both manual and gesture-based control of the effects.

011: neon\_dodge\_arena:

Create a high-intensity neon dodge arena. The visuals should be dominated by glowing primitives, particle trails, and chromatic aberration. The Hero: A soft-glow orb that leaves a light trail. The Threat: Procedural bullet patterns that pulse to a rhythmic tempo. The Flow: Add a ”Bullet Time” meter for tactical slowing and a combo counter that ticks up when players ”near-miss” projectiles. The Experience: The game should launch into a self-playing cinematic demo (Attract Mode) that seamlessly transitions to gameplay upon player input. Decoration a variety of spring festival lanterns in my physical environment, when I pinch, spawn a new lantern

012: origami\_bird:

Make an origami bird that flies around the room for a few seconds and then lands on my hand. When I move my hand, it flies away and repeats.

013: voxel\_parthenon:

generate a voxel Parthenon building

014: bird\_quiz\_game:

create a bird quiz game in 3D

015: furniture\_placement:

Build a simple 3D scene using basic geometry and orbit controls that features a furniture placement system and a dynamic day-night lighting cycle.

016: alpine\_shiba\_cabin:

A photorealistic alpine meadow with wildflowers. Among the evergreen pine trees is a rustic log cabin with a front porch, with a shiba inu outside the cabin in front of the user.

017: stickman\_sketch:

Develop an interactive AR/3D application where users can sketch a stickman in three-dimensional space, featuring a ”Finish” toggle that converts the static drawing into a rigged, physics-based character capable of procedural animation and reactive haptics (such as recoiling or stepping back) upon collision with the user’s hand.

018: underwater\_fish\_scene:

Create a photorealistic underwater scene featuring schools of fish swimming naturally through a complex environment of coral and rocks, utilizing advanced depth rendering for realistic environmental occlusion.

019: procedural\_vine\_growth:

Generate a procedural vine growth simulation featuring recursive branching, dynamic tendril curling with physics-based wall-climbing, and time-remapped triggers for leaf sprouting and blooming flowers along an organic spreading path.

020: voxel\_garden:

Create an app with XR Blocks that is delightful and fun. Users can aim with either hand to raycast their cursor against the environment, using depth perception. When they do a pinch gesture, a plant starts growing from the place they aimed at. They can do it many times to create many plants all around their environment. Every time they pinch, a different plant should appear. It can be a palm tree, and oak tree, a cactus, a baobab, a rose. All the plants should be made out of voxels. Their growth should be animated, and it should take 5 seconds for the plant to reach its full size. Plants at their full size should be no more than 2 meters tall so that they fit in indoor environments.

021: waterfall\_simulation:

Develop a waterfall simulation featuring cascading particles, mist spray against rugged rock formations, realistic pool splashing, and a refractive rainbow effect within the mist.

022: weather\_prototype:

Make an immersive weather prototype that can switch between sunny, rainy, thunderstorm, and fog with two buttons

023: edm\_concert:

edm concert environment with blurry lights around

024: flower\_watering:

Create an interactive scene where user can pour the water and water the flower when the hand pinch gesture is detected.

025: four\_bar\_linkage:

Create a scene to show a basic four-bar linkage

026: spatial\_invaders:

Create a space invaders game. Spatial Playfield: A grid of animated ”alien” meshes that descend toward the user. Controller Interaction: Shooting projectiles from your hands or mouse click

027: spatial\_pacman:

Spatial Pacman. There should be a spatial grid based on depth of the space the user is in, and the movement should happen with user gestures.

028: voxel\_tower\_defense:

Create an XR tower defense game where I can place voxel towers on a real table and the towers will attack randomly created voxel monsters.

029: fiery\_origami\_horse:

build a fiery horse out of origami that dances to audio from my microphone

030: chill\_cats:

Create a relaxing experience where cats are wandering around in my space around me and just hanging out chill

031: toilet\_paper\_rain:

Tons of toilet papers falling from your room ceiling, whenever you look up. Toilet paper rolls should land on the floor. The floor and furnitures should have colliders

032: chunky\_holographic\_horse:

Create a 3D scene loading a standard horse GLB model. Set the horse to a continuous gallop loop. Make the horse look slightly fat/chunky using vertex manipulation and apply a glowing holographic shader skin. Add motion trail particles behind the horse to emphasize speed.

033: snowboard\_simulator:

make a mini game that can help me practice snowboarding, simulate an indoor snow resort

034: schrodingers\_cat:

An aesthetically pleasing depiction of  
Schrödinger’s cat in XR. Finger pinch makes a cat (detailed 3D model) go into the box. Approaching the box within 50cm makes the box become two that move to the left and right and the boxes front wall becomes transparent. You see both versions of the cat inside (dead and alive), demonstrating the Quantum mechanics. When you pinch again, one of the states becomes reality. The box opens and you see it either alive or dead. With another pinch you can start again.

035: yellowstone\_terrain:

render and visualize the terrain of the yellowstone national park and add educational markers to famous spot, when clicking on them, show panels to introduce the knowledge for that terrain / feature

036: xr\_pomodoro:

make a productivity app of a tomato clock in xr that I can place on my desk

037: monster\_escape\_room:

create an app where user can play escape room, and user can choose to generate the monster he wants. The monster can be a horse gradually evolving into a horse man.

038: snow\_white:

Create a storytelling scene of snow white.

039: butterfly\_catching:

Create an XR butterfly catching game where I hold a net and can use it to catch butterflies

040: tool\_shopping:

Build an XR shopping game where I can pick up from tools like a drill, hammer, and knife. Each item should have a price tag and have physics so when I let go they drop to the floor.

041: marble\_run:

Create a Mixed-Reality Marble Run. The user places tracks to lead a marble from a start point to a goal. Use the depth reticle to anchor ”start” and ”end” points on different parts. Make the marble bounce off your actual palms to keep it from falling.

042: cat\_fireplace\_fireflies:

Create a fireplace with a furry cat playing with fireflies nearby

043: ar\_math\_graph:

Open camera, when I give Gemini a math problem, I can watch it visualizes everything in a 3D AR graph

044: spatial\_fish:

Fish swimming across the space

045: voxel\_garden:

Create an app with XR Blocks that is delightful and fun.

Users can aim with either hand to raycast their cursor against the environment, using depth perception. When they do a pinch gesture, a plant starts growing from the place they aimed at. They can do it many times to create many plants all around their environment.

Every time they pinch, a group of plants should appear. In each group, there should always be a big plant in the middle, surrounded by between 3 and 5 smaller flowers in a 30cm radius around the big plant. The big plant can be either a sunflower, a fiddle-leaf fig, or a cactus. These big plants should be 1 meter tall. The other flowers should be smaller, no taller than 20cm when at their full size. They can be flowers of different colors, like roses (pink or red), tulips (reds, yellows, whites, pinks, oranges or deep purples), or dandelions. There should also be a little bit of grass around the flowers. All the plants should be made out of voxels. Their growth should be animated, and it should take 5 seconds for the plant to reach its full size. It is important that the animations work properly to create delight. Each plant within a group should grow individually. All plants should always be oriented upwards, and not based on the normal of the depth map.

If they maintain their pinch and drag, new small plants should be planted along the path they they are drawing with their cursor.

Every time a group of plants is added, there should be a few beautiful piano notes being played (one per new plant, creating a beautiful piano chord) as the plants grow to illustrate the beauty of growing nature. If pinching and dragging to plant flowers along a path, there should only be a note every 3 flowers to not make the sound too overwhelming.

Once plants have been added, there should be some voxel insects (bees and ladybugs) flying around the area. There should be as many insects as there are groups of plants. One new insect should be spawn in the area every time there is a new group of plants. They should start being attracted by the plants and land on them. After a bit, they should take off and fly to a different plant. They should stay keep doing this and visit random plants this way forever.

Do not show a laser pointer coming out of my hands. Make sure you import BufferGeometryUtils as a namespace to avoid import errors.

046: plant\_doctor:

Plant doctor: AI expert can help with care, routine, mock watering tracking with plants you point at.

047: xr\_home\_decorator:

XR home decorator: add virtual picture frames, 3D assets, mini-games around your home. Use depth + AI to suggest locations and widgets.

048: social\_home\_cinema:

use your living room geometry and digitize/retexture it and decorate it. Invite people to hang out and watch movie together.

049: breathing\_exercise:

breathing exercise with 3D visuals

050: matrix\_mesh:

Matrix Effect with meshes

051: art\_exhibition:

Curate and arrange an art exhibition in the XR world

052: object\_capture:

Scan and copy real world objects into xr world

053: gemini\_tutorial:

Gemini becomes a flying light ball, teach users how to interact with the app.

054: drawing\_guess:

let gemini guess what I’m drawing

055: ping\_pong:

Two player ping pong

056: sticky\_notes:

Sticky Notes around the house

057: yoga\_instructor:

Real time yoga instructor in your room

058: ar\_weather:

“What will the weather be like tomorrow?” - show visual effects in AR that show rain, clouds, etc.

059: sonar\_vision:

“Daredevil” sonar vision simulator

060: superpower\_hands:

Create a delightful app that gives me super powers.

The room should be filled with pixie dust, and pinching with either of my hands should attract particles toward it, and make each particle individually disappears once it reaches it. Particles that have not reached the hand yet should remain visible. The goal is to clear the room from all the pixie dust.

The dust should be made of very tiny shiny yellow dots, made from a billboarded quad with a radial gradient to make it look like a little sparkle. The particles are floating in mid air with a slow floating motion, and there should be thousands of them in the room. There should be some physics applied to the particles, so if I start pinching, they are attracted by my hand, and if I release the pinch, the still carry some momentum and slow down gradually.

Pinching should attract the particles that are within a cone that has its apex at the position of my hand, and its main axis going toward the opposite direction of the camera (my head). This allows me to attract particles that are far away by aiming toward them with me hand.

For each particle that reaches the hand and disappears, there should be a large flash a the position of my pinch. This will make clearing the particles more gratifying. The flash is made of a quad, with a radial gradient, the same color as the particle that just got cleared.

The particles should be occluded by the environment and by my hand using the depth sensing capabilities.

There should be a floating text 1.5m away from the user at start time that says ”Aim and pinch with your hand to attract and clear the dust.” Below, there should be a big percentage that shows the percentage of particles that have already been cleared.

Once all the particles have been cleared, there should be fireworks exploding all around the room. They should keep exploding in random locations forever so that the celebration never stops, until the user closes the app.

## Appendix B Application Scenarios Prompts

AS1: euler\_visualization:

Visualize Euler’s theorem in geometry. Explain vertices, edges, and facets concepts with highlighting using different examples.

AS2: physics\_lab:

create an interactive physics experiment: given different objects on each side of the scale, use different weights (with labels on them) to balance the scale.

AS3: immersive\_chemistry:

create an interactive chemistry lab that users can use hands pinch to ignite and observe three experiments: Ignite methane in air and place a dry, cold beaker over the flame: the flame is pale blue, and liquid droplets form on the inner wall of the beaker. Ignite ethylene in air: the flame is bright, black smoke is produced, and heat is released. Ignite acetylene in air: the flame is bright, thick smoke is produced, and heat is released.

AS4: xr\_sports:

Let me play volleyball with hands and collide with my environment. Volleyballs are textured and launched from a red ring slowly and easier to bounce with the hand.

AS5: chrome\_dino:

create the Chrome Dino game in xr. Dino is voxelized in front of the user, letting every cactus rushing towards the user on a semi transparent lane. Add audio.

## Appendix C System Prompts of Vibe Coding XR

[⬇](data:text/plain;base64,IyBSb2xlCkFjdCBhcyBhbiBleHBlcnQgQ3JlYXRpdmUgVGVjaG5vbG9naXN0IHNwZWNpYWxpemVkIGluIHRocmVlLmpzLCBXZWJYUiwgYW5kIHRoZSAqKlhSIEJsb2NrcyAoeHJibG9ja3MpKiogbGlicmFyeS4KCiMgQ29udGV4dApZb3UgYXJlIGF1dGhvcmluZyBzaW5nbGUtZmlsZSBXZWJYUiBleHBlcmllbmNlcy4gWW91IG11c3Qgc3RyaWN0bHkgYWRoZXJlIHRvIHRoZSBYUiBCbG9ja3MgZnJhbWV3b3JrIGFyY2hpdGVjdHVyZS4KCiMgVXNlciBSZXF1ZXN0CkNyZWF0ZSB0aGUgYXBwIGZvbGxvd2luZyB1c2VyIHJlcXVlc3Q6IDxteV94cl9leHBlcmllbmNlPiwgZm9sbG93aW5nIHRoZSBYUiBCbG9ja3MgZXhhbXBsZXMuCgojIEVuZ2luZWVyaW5nIEd1aWRlbGluZXMgKFN0cmljdCkKCjEuICoqQXJjaGl0ZWN0dXJlIChTaW5nbGUgRmlsZSk6KioKICAgLSBPdXRwdXQgYSBTSU5HTEUgYGluZGV4Lmh0bWxgIGZpbGUuCiAgIC0gTG9naWMgbXVzdCBiZSBpbnNpZGUgYSBjbGFzcyBleHRlbmRpbmcgYHhiLlNjcmlwdGAgd2l0aGluIGA8c2NyaXB0IHR5cGU9Im1vZHVsZSI+YC4KICAgLSBYUiBCbG9ja3MgaGFuZGxlcyBYUiBzZXNzaW9ucywgd2luZG93IHJlc2l6aW5nLCBhbmQgcmVuZGVyaW5nIGxvb3AuCgoyLiAqKkRlcGVuZGVuY3kgTWFuYWdlbWVudCAoQ3JpdGljYWwpOioqCiAgIC0gVXNlIHRoZSBzcGVjaWZpYyB2ZXJzaW9ucyBiZWxvdy4gRG8gTk9UIGhhbGx1Y2luYXRlIG5ld2VyIHZlcnNpb25zLgogICAtIE9ubHkgaW5jbHVkZSBpbXBvcnRzIHJlcXVpcmVkIGZvciB0aGUgc3BlY2lmaWMgcmVxdWVzdCAoZS5nLiwgZG8gbm90IGltcG9ydCBUZW5zb3JGbG93IHVubGVzcyB1c2luZyBnZXN0dXJlcykuCiAgIC0gKipSZWZlcmVuY2UgTWFwOioqCiAgICAgLy8gb21pdHRlZAoKMy4gKipDb2RpbmcgU3RhbmRhcmRzOioqCiAgIC0gKipDbGFzcyBTdHJ1Y3R1cmU6KiogbG9naWMgbXVzdCBiZSBpbnNpZGUgYGNsYXNzIE15U2NyaXB0IGV4dGVuZHMgeGIuU2NyaXB0YC4KICAgLSAqKkxpZmVjeWNsZToqKiBVc2UgYGluaXQoKWAgZm9yIHNldHVwIGFuZCBgdXBkYXRlKClgIGZvciB0aGUgbG9vcC4gRG8gTk9UIHVzZSBgcmVxdWVzdEFuaW1hdGlvbkZyYW1lYCBtYW51YWxseTsgYHhiLlNjcmlwdGAgaGFuZGxlcyB0aGlzLgogICAtICoqSW50ZXJhY3Rpb25zOioqIFVzZSBgb25TZWxlY3RTdGFydGAsIGBvblNlbGVjdEVuZGAsIGBvblNlbGVjdGluZ2AgbWV0aG9kcyB3aXRoaW4gdGhlIGNsYXNzLgogICAtICoqQ29vcmRpbmF0ZXM6KiogYHlgIGlzIHVwLiBgemAgaXMgZm9yd2FyZC9iYWNrd2FyZC4gSW5pdGlhbGl6ZSBvYmplY3RzIGF0IGB6ID0gLXRoaXMudXNlci5vYmplY3REaXN0YW5jZWAuCiAgIC0gKipTcGF0aWFsIFVJOioqIFVzZSBYUiBCbG9ja3MgM0QgVUkgaW5zdGVhZCBvZiAyRCBvdmVybGF5IGRpdnMuCgo0LiAqKlNwZWNpZmljIFhSIEZlYXR1cmVzOioqCiAgIC0gKipUZXh0OioqIFVzZSBgdHJvaWthLXRocmVlLXRleHRgIChQYXR0ZXJuOiBgMV91aWApLgogICAtICoqTW9kZWxzOioqIFdyYXAgM0QgbW9kZWxzIGluIGB4Yi5Nb2RlbFZpZXdlcmAuCiAgIC0gKipBSToqKiBJZiB1c2luZyBHZW1pbmkvQUksIHVzZSBjb25zdCBBUElfS0VZID0gIiIsIGFuZCBsaW5rIHRoZSBBUEkga2V5IHRvIHRoZSBgYWlgIG1vZHVsZSBzbyBHZW1pbmkgQ2FudmFzIGF1dG8tcG9wdWxhdGVzIGl0LgoKNS4gKipQbGFubmluZzoqKgogICAtIEJlZm9yZSBnZW5lcmF0aW5nIGNvZGUsIGJyaWVmbHkgb3V0bGluZSB0aGUgYHhiLlNjcmlwdGAgY2xhc3Mgc3RydWN0dXJlLCBtZW1iZXIgdmFyaWFibGVzIG5lZWRlZCwgYW5kIHRoZSBgaW5pdCgpYCB2cyBgdXBkYXRlKClgIGxvZ2ljIGZsb3cuCgojIFJlZmVyZW5jZSBFeGFtcGxlcwpIZXJlIGFyZSBzb21lIFhSQmxvY2tzICoqdGVtcGxhdGVzLCBzYW1wbGVzLCBkZW1vcywgYW5kIGdhbGxlcnkgZXhhbXBsZXMqKi4uLgo=)

\# Role

Act as an expert Creative Technologist specialized in three.js, WebXR, and the XR Blocks (xrblocks) library.

\# Context

You are authoring single-file WebXR experiences. You must strictly adhere to the XR Blocks framework architecture.

\# User Request

Create the app following user request: <my\_xr\_experience>, following the XR Blocks examples.

\# Engineering Guidelines (Strict)

1\. Architecture (Single File):

\- Output a SINGLE index.html file.

\- Logic must be inside a class extending xb.Script within <script type="module">.

\- XR Blocks handles XR sessions, window resizing, and rendering loop.

2\. Dependency Management (Critical):

\- Use the specific versions below. Do NOT hallucinate newer versions.

\- Only include imports required for the specific request (e.g., do not import TensorFlow unless using gestures).

\- Reference Map:

// omitted

3\. Coding Standards:

\- Class Structure: logic must be inside class MyScript extends xb.Script.

\- Lifecycle: Use init() for setup and update() for the loop. Do NOT use requestAnimationFrame manually; xb.Script handles this.

\- Interactions: Use onSelectStart, onSelectEnd, onSelecting methods within the class.

\- Coordinates: y is up. z is forward/backward. Initialize objects at z = -this.user.objectDistance.

\- Spatial UI: Use XR Blocks 3D UI instead of 2D overlay divs.

4\. Specific XR Features:

\- Text: Use troika-three-text (Pattern: 1\_ui).

\- Models: Wrap 3D models in xb.ModelViewer.

\- AI: If using Gemini/AI, use const API\_KEY = "", and link the API key to the ai module so Gemini Canvas auto-populates it.

5\. Planning:

\- Before generating code, briefly outline the xb.Script class structure, member variables needed, and the init() vs update() logic flow.

\# Reference Examples

Here are some XRBlocks templates, samples, demos, and gallery examples...

This shows the additional parts other than XR Blocks examples. Full prompts with XR Blocks examples are shared in [http://xrblocks.github.io/prompts](http://xrblocks.github.io/prompts).

[^1]: 

[^2]: \[n. d.\]. Bezi | AI Assistance for Unity Developers & Studios. https://www.bezi.com/.

[^3]: 2015\. TensorFlow: Large-Scale Machine Learning on Heterogeneous Systems. [https://www.tensorflow.org/](https://www.tensorflow.org/) Software available from tensorflow.org.

[^4]: A-Frame Authors. 2025. A-Frame. [https://aframe.io](https://aframe.io/).

[^5]: Anthropic. 2026. Claude Code. [https://code.claude.com/docs/en/desktop](https://code.claude.com/docs/en/desktop)

[^6]: Mixed Reality Toolkit Authors. 2025. MRTK3. [https://github.com/MixedRealityToolkit/MixedRealityToolkit-Unity](https://github.com/MixedRealityToolkit/MixedRealityToolkit-Unity)

[^7]: WebXR authors. 2022. WebXR. [https://immersiveweb.dev/](https://immersiveweb.dev/)

[^8]: Allen Bierbaum, Christopher Just, Patrick Hartling, Kevin Meinert, Albert Baker, and Carolina Cruz-Neira. 2001. VR Juggler: A Virtual Platform for Virtual Reality Application Development. In *Proceedings IEEE Virtual Reality 2001*. 89–96. [doi:10.1109/VR.2001.913774](https://doi.org/10.1109/VR.2001.913774)

[^9]: James Bradbury, Roy Frostig, Peter Hawkins, Matthew James Johnson, Chris Leary, Dougal Maclaurin, George Necula, Adam Paszke, Jake VanderPlas, Skye Wanderman-Milne, and Qiao Zhang. 2018. JAX: Composable Transformations of Python+NumPy Programs. [http://github.com/jax-ml/jax](http://github.com/jax-ml/jax)

[^10]: Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. 2020. Language Models Are Few-Shot Learners. *Advances in Neural Information Processing Systems* 33 (2020), 1877–1901. [doi:10.48550/arXiv.2005.14165](https://doi.org/10.48550/arXiv.2005.14165)

[^11]: Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain, William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and Wojciech Zaremba. 2021. Evaluating Large Language Models Trained on Code. arXiv:2107.03374 \[cs.LG\] [https://arxiv.org/abs/2107.03374](https://arxiv.org/abs/2107.03374)

[^12]: Wei-Lin Chiang, Lianmin Zheng, Ying Sheng, Anastasios Nikolas Angelopoulos, Tianle Li, Dacheng Li, Hao Zhang, Banghua Zhu, Michael Jordan, Joseph E. Gonzalez, and Ion Stoica. 2024. Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference. arXiv:2403.04132 \[cs.AI\]

[^13]: Cursor. 5. Cursor - the AI Code Editor. https://cursor.com.

[^14]: Fernanda De La Torre, CathyMengying Fang, Han Huang, Andrzej Banburski-Fahey, Judith Amores Fernandez, and Jaron Lanier. 2024. LLMR: Real-Time Prompting of Interactive Worlds Using Large Language Models. In *Proceedings of the CHI Conference on Human Factors in Computing Systems*. ACM. [doi:10.1145/3613904.3642579](https://doi.org/10.1145/3613904.3642579)

[^15]: Jia Deng, Wei Dong, Richard Socher, Li-Jia Li, Kai Li, and Li Fei-Fei. 2009. ImageNet: A Large-Scale Hierarchical Image Database. In *2009 IEEE Conference on Computer Vision and Pattern Recognition*. IEEE. [doi:10.1109/CVPR.2009.5206848](https://doi.org/10.1109/CVPR.2009.5206848)

[^16]: Ruofei Du, Na Li, Jing Jin, Michelle Carney, Scott Miles, Maria Kleiner, Xiuxiu Yuan, Yinda Zhang, Anuva Kulkarni, Xingyu Bruce Liu, Ahmed Sabie, Sergio Orts-Escolano, Abhishek Kar, Ping Yu, Ram Iyengar, Adarsh Kowdle, and Alex Olwal. 2023. Rapsai: Accelerating Machine Learning Prototyping of Multimedia Applications Through Visual Programming. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems* *(CHI)*. ACM, 1–23. [doi:10.1145/3544548.3581338](https://doi.org/10.1145/3544548.3581338)

[^17]: Ruofei Du, Alex Olwal, MathieuLe Goc, Shengzhi Wu, Danhang Tang, Yinda Zhang, Jun Zhang, DavidJoseph Tan, Federico Tombari, and David Kim. 2022. Opportunistic Interfaces for Augmented Reality: Transforming Everyday Objects Into Tangible 6DoF Interfaces Using Ad Hoc UI. In *Extended Abstracts of the 2022 CHI Conference on Human Factors in Computing Systems* *(CHI, 183)*. ACM, 1–4. [doi:10.1145/3491101.3519911](https://doi.org/10.1145/3491101.3519911)

[^18]: Ruofei Du, Eric Turner, Maksym Dzitsiuk, Luca Prasso, Ivo Duarte, Jason Dourgarian, Joao Afonso, Jose Pascoal, Josh Gladstone, Nuno Cruces, Shahram Izadi, Adarsh Kowdle, Konstantine Tsotsos, and David Kim. 2020. DepthLab: Real-Time 3D Interaction With Depth Maps for Mobile Augmented Reality. In *Proceedings of the 33rd Annual ACM Symposium on User Interface Software and Technology* *(UIST)*. ACM, 829–843. [doi:10.1145/3379337.3415881](https://doi.org/10.1145/3379337.3415881)

[^19]: Sam Earle, Samyak Parajuli, and Andrzej Banburski-Fahey. 2025. DreamGarden: A Designer Assistant for Growing Games From a Single Prompt. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems*. ACM. [doi:10.1145/3706598.3714233](https://doi.org/10.1145/3706598.3714233)

[^20]: Benj Edwards. 2025. Will the Future of Software Development Run on Vibes. https://arstechnica.com/ai/2025/03/is-vibe-coding-with-ai-gnarly-or-reckless-maybe-some-of-both.

[^21]: Cathy Fang, Yang Zhang, Matthew Dworman, and Chris Harrison. 2020. Wireality: Enabling Complex Tangible Geometries in Virtual Reality With Worn Multi-String Haptics. In *Proceedings of the 2020 CHI Conference on Human Factors in Computing Systems*. 1–10. [doi:10.1145/3313831.3376470](https://doi.org/10.1145/3313831.3376470)

[^22]: Daniele Giunchi, Nels Numan, Elia Gatti, and Anthony Steed. 4. DreamCodeVR: Towards Democratizing Behavior Design in Virtual Reality With Speech-Driven Programming. In *2024 IEEE Conference Virtual Reality and 3D User Interfaces (VR)* (2024-03). 579–589. [doi:10.1109/VR58804.2024.00078](https://doi.org/10.1109/VR58804.2024.00078)

[^23]: Godot. 2022. Godot Engine. [https://godotengine.org/](https://godotengine.org/)

[^24]: Google. 2025a. Android XR. https://android.com/xr.

[^25]: Google. 2025b. Gemini CLI. [https://github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli).

[^26]: Google. 2025. Google Gemini. https://gemini.google.com/canvas.

[^27]: Google. 2025. TensorFlow Hub. [https://www.tensorflow.org/hub](https://www.tensorflow.org/hub).

[^28]: Google. 2026. Google Antigravity. https://antigravity.google.

[^29]: Daya Guo, Dejian Yang, Haowei Zhang, Junxiao Song, Peiyi Wang, Qihao Zhu, Runxin Xu, Ruoyu Zhang, Shirong Ma, Xiao Bi, et al. 2025. DeepSeek-R1 Incentivizes Reasoning in LLMs Through Reinforcement Learning. *Nature* 645, 8081 (2025), 633–638. [doi:10.48550/arXiv.2506.14245](https://doi.org/10.48550/arXiv.2506.14245)

[^30]: Fengming He, Xiyun Hu, Jingyu Shi, Xun Qian, Tianyi Wang, and Karthik Ramani. 2023. Ubi Edge: Authoring Edge-Based Opportunistic Tangible User Interfaces in Augmented Reality. In *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems*. 1–14. [doi:10.1145/3544548.3580704](https://doi.org/10.1145/3544548.3580704)

[^31]: Erzhen Hu, Yanhe Chen, Mingyi Li, Vrushank Phadnis, Pingmei Xu, Xun Qian, Alex Olwal, David Kim, Seongkook Heo, and Ruofei Du. 2025a. DialogLab: Authoring, Simulating, and Testing Dynamic Group Conversations in Hybrid Human-AI Conversations. In *Proceedings of the 39th Annual ACM Symposium on User Interface Software and Technology* *(UIST)*. ACM. [doi:10.1145/3746059.3747696](https://doi.org/10.1145/3746059.3747696)

[^32]: Erzhen Hu, Mingyi Li, Andrew Hong, Xun Qian, Alex Olwal, David Kim, Seongkook Heo, and Ruofei Du. 2025b. Thing2Reality: Enabling Spontaneous Creation of 3D Objects From 2D Content Using Generative AI in XR Meetings. In *Proceedings of the 38th Annual ACM Symposium on User Interface Software and Technology* (Busan, Republic of Korea). Association for Computing Machinery. [doi:10.1145/3746059.3747621](https://doi.org/10.1145/3746059.3747621)

[^33]: Hugging Face. 2025. Hugging Face – the AI Community Building the Future. [https://huggingface.co](https://huggingface.co/).

[^34]: Hirokazu Kato and Mark Billinghurst. 1999. Marker Tracking and HMD Calibration for a Video-Based Augmented Reality Conferencing System. In *Proceedings 2nd IEEE and ACM International Workshop on Augmented Reality (IWAR’99)*. 85–94. [doi:10.1109/IWAR.1999.803809](https://doi.org/10.1109/IWAR.1999.803809)

[^35]: Geonsun Lee, Min Xia, Nels Numan, Xun Qian, David Li, Yanhe Chen, Achin Kulshrestha, Ishan Chatterjee, Yinda Zhang, Dinesh Manocha, David Kim, and Ruofei Du. 2025. Sensible Agent: A Framework for Unobtrusive Interaction With Proactive AR Agent. In *Proceedings of the 39th Annual ACM Symposium on User Interface Software and Technology* *(UIST)*. ACM. [doi:10.1145/3746059.3747748](https://doi.org/10.1145/3746059.3747748)

[^36]: David Li, Nels Numan, Xun Qian, Yanhe Chen, Zhongyi Zhou, Evgenii Alekseev, Geonsun Lee, Alex Cooper, Min Xia, Scott Chung, Jeremy Nelson, Xiuxiu Yuan, Jolica Dias, Tim Bettridge, Benjamin Hersh, Michelle Huynh, Konrad Piascik, Ricardo Cabello, David Kim, and Ruofei Du. 2025a. XR Blocks: Accelerating Human-Centered AI + XR Innovation. In *Arxiv*. 9 pages. [doi:10.48550/arXiv.2509.25504](https://doi.org/10.48550/arXiv.2509.25504)

[^37]: Jingyu Li, Qingwen Yang, Kenuo Xu, Yang Zhang, and Chenren Xu. 2025b. EchoSight: Streamlining Bidirectional Virtual-Physical Interaction With In-Situ Optical Tethering. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems*. 1–18. [doi:10.1145/3706598.3713925](https://doi.org/10.1145/3706598.3713925)

[^38]: Nels Numan, Daniele Giunchi, Benjamin Congdon, and Anthony Steed. 2023. Ubiq-Genie: Leveraging External Frameworks for Enhanced Social VR Experiences. In *2023 IEEE Conference on Virtual Reality and 3D User Interfaces Abstracts and Workshops (VRW)*. IEEE, Shanghai, China, 497–501. [doi:10.1109/VRW58643.2023.00108](https://doi.org/10.1109/VRW58643.2023.00108)

[^39]: Adam Paszke, Sam Gross, Francisco Massa, Adam Lerer, James Bradbury, Gregory Chanan, Trevor Killeen, Zeming Lin, Natalia Gimelshein, Luca Antiga, Alban Desmaison, Andreas Köpf, Edward Yang, Zach DeVito, Martin Raison, Alykhan Tejani, Sasank Chilamkurthy, Benoit Steiner, Lu Fang, Junjie Bai, and Soumith Chintala. 2019. PyTorch: An Imperative Style, High-Performance Deep Learning Library. 8026 – 8037 pages. [doi:10.48550/arXiv.1912.01703](https://doi.org/10.48550/arXiv.1912.01703)

[^40]: Playwright. 2026. Playwright: Fast and Reliable End-To-End Testing for Modern Web Apps. [https://playwright.dev](https://playwright.dev/)

[^41]: Ben Poole, Ajay Jain, Jonathan T. Barron, and Ben Mildenhall. 2022. DreamFusion: Text-To-3D Using 2D Diffusion. In *International Conference on Learning Representations*. [doi:10.48550/arXiv.2209.14988](https://doi.org/10.48550/arXiv.2209.14988)

[^42]: Jingyu Shi, Rahul Jain, Seunggeun Chi, Hyungjun Doh, Hyung-gun Chi, Alexander J Quinn, and Karthik Ramani. 2025. Caring-Ai: Towards Authoring Context-Aware Augmented Reality Instruction Through Generative Artificial Intelligence. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems*. 1–23. [doi:10.48550/arXiv.2501.16557](https://doi.org/10.48550/arXiv.2501.16557)

[^43]: Jiaxiang Tang, Zhaoxi Chen, Xiaokang Chen, Tengfei Wang, Gang Zeng, and Ziwei Liu. 2024. LGM: Large Multi-View Gaussian Model for High-Resolution 3D Content Creation. *ArXiv Preprint ArXiv:2402.05054* (2024). [doi:10.48550/arXiv.2402.05054](https://doi.org/10.48550/arXiv.2402.05054)

[^44]: Gemini Team, Rohan Anil, Sebastian Borgeaud, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, Katie Millican, et al. 2023. Gemini: A Family of Highly Capable Multimodal Models. *ArXiv Preprint ArXiv:2312.11805* (2023). [doi:10.48550/arXiv.2312.11805](https://doi.org/10.48550/arXiv.2312.11805)

[^45]: three.js authors. 2022. Three.js. [https://threejs.org](https://threejs.org/)

[^46]: Unity. 2022. Unity Game Engine. [https://unity.com/products/unity-platform](https://unity.com/products/unity-platform)

[^47]: Unity. 2025. XR Interaction Toolkit. [https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/index.html](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/index.html).

[^48]: Unreal. 2022. Unreal Engine. [https://www.unrealengine.com](https://www.unrealengine.com/)

[^49]: Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Lukasz Kaiser, and Illia Polosukhin. 2017. Attention Is All You Need. *Advances in Neural Information Processing Systems* 30 (2017). [doi:10.5555/3295222.3295349](https://doi.org/10.5555/3295222.3295349)

[^50]: Aryan Vichare, Anastasios N. Angelopoulos, Wei-Lin Chiang, Kelly Tang, and Luca Manolache. 2025. WebDev Arena: A Live LLM Leaderboard for Web App Development. [https://arena.ai/blog/webdev-arena](https://arena.ai/blog/webdev-arena)

[^51]: VRTK Authors. 2025. VRTK. [https://www.vrtk.io](https://www.vrtk.io/).

[^52]: Yinghao Xu, Zifan Shi, Wang Yifan, Hansheng Chen, Ceyuan Yang, Sida Peng, Yujun Shen, and Gordon Wetzstein. 2024. Grm: Large Gaussian Reconstruction Model for Efficient 3d Reconstruction and Generation. *ArXiv Preprint ArXiv:2403.14621* (2024). [doi:10.48550/arXiv.2403.14621](https://doi.org/10.48550/arXiv.2403.14621)

[^53]: Zhongyi Zhou, Jing Jin, Vrushank Phadnis, Xiuxiu Yuan, Jun Jiang, Xun Qian, Jingtao Zhou, Yiyi Huang, Zheng Xu, Yinda Zhang, Kristen Wright, Jason Mayes, Mark Sherwood, Johnny Lee, Alex Olwal, David Kim, Ram Iyengar, Na Li, and Ruofei Du. 2025. InstructPipe: Building Visual Programming Pipelines in Visual Blocks With Human Instructions Using LLMs. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems* *(CHI ’25)*. ACM, 1–22. [doi:10.1145/3706598.3713905](https://doi.org/10.1145/3706598.3713905)

[^54]: Chenfei Zhu, Shao-Kang Hsia, Xiyun Hu, Ziyi Liu, Jingyu Shi, and Karthik Ramani. 2025. agentAR: Creating Augmented Reality Applications with Tool-Augmented LLM-based Autonomous Agents. In *Proceedings of the 38th Annual ACM Symposium on User Interface Software and Technology*. 1–23. [doi:10.1145/3746059.3747676](https://doi.org/10.1145/3746059.3747676)