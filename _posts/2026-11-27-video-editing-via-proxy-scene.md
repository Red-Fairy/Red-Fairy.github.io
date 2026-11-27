---
layout: research-note
title: Video editing through an agent-built proxy scene
description: An agent rebuilds a clip as a 3D scene in code, edits it there, and a video diffusion model turns the untextured render back into video. One framework covers camera, time and visual effects, and every edit stays adjustable afterwards.
topic: Video editing
thumbnail: /assets/img/blog/proxy-scene.jpg
---
{% assign V = '/assets/video/proxy-scene/' | relative_url %}
<style>
  /* This note only: one column for text and figures, and a small colour system (blue agent, violet edit, slate proxy, orange diffusion). */
  .note { max-width: 56rem; --c-agent: #155fbe; --c-edit: #6d3fc4; --c-proxy: #4b5563; --c-diff: #c2570c; --c-good: #0b7a55;
          --t-agent: #e8f0fe; --t-edit: #f0eafc; --t-proxy: #eef0f3; --t-diff: #fdeede; --t-good: #e3f4ec; }
  .note-heading h1 { background: linear-gradient(90deg, var(--c-agent), var(--c-edit) 55%, var(--c-diff)); -webkit-background-clip: text; background-clip: text; color: transparent; }
  .prose h2 { margin-top: 3.2rem; padding: .1rem 0 .1rem .8rem; border-left: .32rem solid var(--c-agent); }
  .prose h3 { color: var(--c-agent); }
  .prose code { background: var(--t-agent); color: #0d3c7a; padding: .08em .35em; border-radius: .25rem; font-size: .9em; }
  .prose .wide { width: auto; }
  .prose .panel { margin: 1.8rem 0; padding: .7rem; background: var(--surface); border: 1px solid var(--rule); border-radius: .7rem; }
  .prose .clips { display: grid; gap: .6rem; margin: 0; }
  .prose .clips.two { grid-template-columns: repeat(auto-fit, minmax(min(100%, 24rem), 1fr)); }
  .prose .clips figure { margin: 0; }
  .prose video { display: block; width: 100%; height: auto; border-radius: .4rem; background: #111; }
  .prose .clips figcaption { margin-top: .3rem; font-size: .875rem; color: var(--body); }
  .prose .cap { margin: .7rem .2rem .1rem; padding-left: .7rem; border-left: 3px solid var(--c-edit); color: var(--muted); font-size: .9375rem; }
  .prose .callout { margin: 1.4rem 0 1.8rem; padding: 1rem 1.2rem; background: linear-gradient(135deg, var(--t-agent), var(--t-edit)); border-radius: .7rem; color: var(--ink); }
  .prose .callout b { color: var(--c-agent); }
  .prose .cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: .7rem; margin: 1.6rem 0; }
  .prose .card { padding: .9rem 1rem; border-radius: .6rem; font-size: .98rem; line-height: 1.55; border-top: .28rem solid; }
  .prose .card b { display: block; margin-bottom: .25rem; color: var(--ink); font-size: 1.05rem; }
  .prose .card.a { background: var(--t-agent); border-color: var(--c-agent); }
  .prose .card.b { background: var(--t-edit); border-color: var(--c-edit); }
  .prose .card.c { background: var(--t-good); border-color: var(--c-good); }
  .prose .pipeline { display: grid; grid-template-columns: repeat(4, 1fr); gap: 1.4rem; margin: 1.8rem 0 .8rem; padding: 0; list-style: none; counter-reset: step; }
  .prose .pipeline li { position: relative; counter-increment: step; padding: .9rem 1rem 1rem; border-radius: .6rem; font-size: .95rem; line-height: 1.5; border-top: .28rem solid var(--c); background: var(--t); }
  .prose .pipeline li::before { content: counter(step); display: inline-grid; place-items: center; width: 1.6rem; height: 1.6rem; margin-bottom: .4rem; border-radius: 50%; background: var(--c); color: #fff; font-weight: 700; font-size: .9rem; }
  .prose .pipeline li:not(:last-child)::after { content: "\2192"; position: absolute; right: -1.15rem; top: 50%; transform: translateY(-50%); color: var(--muted); font-size: 1.2rem; }
  .prose .pipeline b { display: block; color: var(--ink); font-size: 1.02rem; }
  .prose .pipeline .s1 { --c: var(--c-agent); --t: var(--t-agent); } .prose .pipeline .s2 { --c: var(--c-edit); --t: var(--t-edit); }
  .prose .pipeline .s3 { --c: var(--c-proxy); --t: var(--t-proxy); } .prose .pipeline .s4 { --c: var(--c-diff); --t: var(--t-diff); }
  .prose table { display: table; width: 100%; margin: 1.6rem 0; font-size: 1rem; border-radius: .5rem; overflow: hidden; }
  .prose th { background: var(--t-agent); color: #0d3c7a; font-weight: 600; }
  .prose tbody tr:nth-child(even) td { background: var(--surface); }
  .prose ul.limits { padding: 1rem 1.2rem 1rem 2.4rem; background: var(--t-diff); border-radius: .7rem; }
  .prose ul.limits li::marker { color: var(--c-diff); }
  .prose ul.limits strong { color: #8a3d06; }
  @media (max-width: 46rem) {
    .prose .cards { grid-template-columns: 1fr; }
    .prose .pipeline { grid-template-columns: 1fr 1fr; gap: 1rem; }
    .prose .pipeline li:not(:last-child)::after { content: none; }
    .prose table { display: block; overflow-x: auto; }
  }
  @media (max-width: 26rem) { .prose .pipeline { grid-template-columns: 1fr; } }
</style>

<div class="callout"><b>In short.</b> Most controllable video editing trains a dedicated model for each kind of control: one for camera, one for time, one for pose, one for effects. We take a different route. A coding agent rebuilds the clip as a 3D scene written in Blender Python. The edit is made in that scene, in code. A video diffusion model then turns an untextured grey render of the edited scene into a photorealistic video. The same three steps give camera control, time control and visual effects, with no model trained for any of them, and an edit can be changed afterwards by changing a number.</div>

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}param_ufo.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}param_ufo_proxy.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">One effect script, three settings. An agent built "a flying saucer abducts the car" once. The second and third columns were made without an agent by re-running its script with <code>START</code> 8 frames later and with <code>STRENGTH</code> 0.5, where the car is lifted half way and set back down. Top: final videos. Bottom: the grey proxy renders they were made from.</p>
</div>

## Introduction

Video diffusion models are now good enough that generated footage of people is hard to tell from real footage. Controlling them is a separate problem. The usual answer is a specialised model per control signal: a camera-controlled model, a retiming model, a pose-driven model, an effects model. Each needs its own training data and its own training run, and each covers only its own kind of edit.

Two things make a different design possible today.

- **Agents can write scenes.** A capable vision-language agent with a code interpreter can watch a short clip and reconstruct it as a 3D scene in code: the set, a calibrated camera, and a rigged, animated subject. Code is a convenient representation for editing, because moving a camera, retiming an animation or adding an object are all ordinary program changes.
- **Diffusion models can render.** An agent-built scene has little texture and simple shading, and some things are very hard to build by hand at all. Even the best studio animation is still easy to tell from live action, while diffusion models produce footage that is not, which says their space of images is far larger than what explicit 3D assets reach. So we do not ask the agent for realism. We ask it for geometry and motion, and leave appearance to a diffusion model.

Our pipeline joins the two. The agent's scene is rendered as a grey, untextured **proxy video**. A structure-conditioned video model, SolarRenderer, turns that proxy into the final video, taking the look from the first frame of the original clip and a text prompt.

This has three consequences that we explore below.

<div class="cards">
  <div class="card a"><b>One framework for many edits</b>Camera paths, speed changes and visual effects are all edits to the same scene, rendered by the same model.</div>
  <div class="card b"><b>Consistency across edits</b>Every edit re-renders the same 3D scene, so geometry, layout and motion agree between rounds of editing without any memory mechanism.</div>
  <div class="card c"><b>Editability</b>A generated video is hard to steer: when the result is not what you wanted, you re-roll and hope. Here the effect is a program with named parameters, so "the same effect, eight frames later" or "half as strong" is a parameter change followed by a re-render.</div>
</div>

The cost is speed. The agent takes up to an hour to rebuild a clip, where a specialised model needs one diffusion pass. We come back to this in the limitations.

## Method

<ol class="pipeline">
  <li class="s1"><b>Reconstruct</b>A scene agent watches the clip and writes a Blender scene: set, camera, rigged subject, all animation baked.</li>
  <li class="s2"><b>Edit in 3D</b>A camera path, a retime, or an effect script written by a second agent changes the scene.</li>
  <li class="s3"><b>Render the proxy</b>The edited scene is rendered from the chosen camera as a grey, untextured video.</li>
  <li class="s4"><b>Render the video</b>A diffusion model turns the proxy into video, given the clip's first frame and a prompt.</li>
</ol>

### Reconstruction by an agent

The scene agent is a coding agent (gpt-6.2-astra, run through the Codex CLI in a sandbox) with Blender and a 60-minute budget. It receives the clip's frames and a short brief, and must deliver a Python script that rebuilds the scene from nothing: the set, a camera matched to the footage, and the subject as a rigged mesh whose animation is baked to keyframes. Human pose and image-to-3D estimators are available to it as optional tools; on the clips shown here the agents mostly modelled the subject themselves, for example an eight-bone car with steering and wheel spin fitted to wheel hubs and roof rails in the footage, or a sixteen-bone swan.

The harness replays the script in a clean container and runs checks on the result: motion smoothness, constant bone lengths, intersections with the set, and how much of the frame the set covers from nearby viewpoints.

### Editing in the scene

Because the scene is explicit, three kinds of edit need no learning at all.

- **Camera.** A new camera path is a new camera animation. We use paths that start exactly at the clip's own camera, so the first frame is unchanged: orbits of 90° and 180°, a rise, and a pan away and back.
- **Time and motion.** A speed change re-samples the subject's animation. At speeds above 1× the subject reaches the end of its recorded motion early and then continues along its path, rather than stopping or reversing. For a person with a skeleton the same re-sampling works per person and per body part.
- **Visual effects.** A second agent receives the reconstructed scene and a one-paragraph description of the effect, and writes a script that adds it as mesh and motion only: no particles, volumes or textures, since the proxy shows only geometry. Squashing a car, shattering a bear into ceramic shards, or adding a giant hand that crushes the car are all built this way.

### From proxy to video

The edited scene is rendered with Blender's Workbench engine in a single grey material. SolarRenderer, a fine-tune of the MiniMax-H3 video model, takes three inputs: this proxy video for structure and motion, the first frame of the original clip for appearance, and a text prompt describing the scene and the edit. It was trained once, for the general task of rendering a structure video, and is not adapted to any edit type here.

### Making effects adjustable

The first effect scripts our agents wrote were hard to change afterwards. Frame numbers were written into the code, and some scripts shipped precomputed data tied to one moment of the clip. We reviewed 47 of them: by our reading only about a fifth could be retimed by changing one number.

So we changed what the effect agent must deliver.

- The script opens by declaring named parameters with valid ranges: `START`, `DURATION` and `STRENGTH` always, plus any that suit the effect.
- Nothing below may use a fixed frame number, and nothing may be precomputed. The subject's pose is sampled at the frames the parameters give, and contacts are computed when the script runs.
- A check re-runs the script at its defaults, at a different start, at half strength and at both, and fails if the clip changes before `START`, if nothing happens after it, or if half strength looks the same as full.

After that, a new setting needs no agent: the script is re-run with an override such as `{"START": 34}`, the proxy is rendered again, and the diffusion model produces the video.

## Experiments

Most clips are from DAVIS, between 50 and 104 frames long (2 to 4 seconds); the scenes with people are longer. Everything is rendered at 1344×768. Each result below comes from one agent run and one diffusion seed. Videos loop; in every pair the grey proxy is on the left.

### Reconstruction

Rendering the unedited scene from the clip's own camera shows what the pipeline preserves when nothing is changed.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}recon_car_turn.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_blackswan.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_horsejump_low.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_bear.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_cow.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_bus.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_boat.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}recon_car_roundabout.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">Source clip, the agent's proxy scene from the same camera, and our render of that proxy.</p>
</div>

Against the DAVIS masks, the subject's silhouette in the agent's scene reaches a mean IoU between 0.67 and 0.91 across eleven clips: 0.91 for the car on the mountain road, 0.87 for the swan, 0.83 for the bear. The proxy is a simplification, and the render follows the proxy, so details the agent did not model (fur, foliage, exact gait) come from the diffusion model and can differ from the clip.

### Camera control

<div class="panel">
  <div class="clips two">
    <figure><video src="{{ V }}cam_car_orbitR90.mp4" autoplay muted loop playsinline></video><figcaption>Orbit right 90°</figcaption></figure>
    <figure><video src="{{ V }}cam_car_moveup.mp4" autoplay muted loop playsinline></video><figcaption>Move up</figcaption></figure>
    <figure><video src="{{ V }}cam_car_whip.mp4" autoplay muted loop playsinline></video><figcaption>Pan away and back</figcaption></figure>
    <figure><video src="{{ V }}cam_swan_orbitL180.mp4" autoplay muted loop playsinline></video><figcaption>Orbit left 180°</figcaption></figure>
    <figure><video src="{{ V }}cam_swan_orbitR90.mp4" autoplay muted loop playsinline></video><figcaption>Orbit right 90°</figcaption></figure>
    <figure><video src="{{ V }}cam_horse_orbitL90.mp4" autoplay muted loop playsinline></video><figcaption>Orbit left 90°</figcaption></figure>
  </div>
  <p class="cap">New camera paths through the reconstructed scenes. Every path starts at the clip's own camera.</p>
</div>

The subject keeps its identity from views the clip never shows, such as the swan head-on or the car from behind. When the camera pans away, the model does not invent a second subject in the empty view. Where the new view shows parts of the set that the agent modelled only roughly, the render becomes cleaner and more synthetic than the clip, and an object between camera and subject, such as a tree on a 180° orbit, is rendered as it stands in the proxy.

### Time control

<div class="panel">
  <div class="clips two">
    <figure><video src="{{ V }}time_car_slow.mp4" autoplay muted loop playsinline></video><figcaption>0.25× speed</figcaption></figure>
    <figure><video src="{{ V }}time_car_fast.mp4" autoplay muted loop playsinline></video><figcaption>2× speed: the car drives on past the end of the clip's motion</figcaption></figure>
    <figure><video src="{{ V }}time_swan_slow.mp4" autoplay muted loop playsinline></video><figcaption>0.5× speed</figcaption></figure>
    <figure><video src="{{ V }}time_swan_fast.mp4" autoplay muted loop playsinline></video><figcaption>2× speed</figcaption></figure>
  </div>
  <p class="cap">Retiming the subject. The camera is retimed with it so that it keeps following.</p>
</div>

At 2× the car finishes the clip's motion after 40 frames and then keeps driving along the road the agent surveyed, about 21 metres further, with its wheels still turning. That continuation is possible because the scene contains the road and the car's rig, not just the frames that were filmed.

### Motion control for people

Scenes with people are reconstructed the same way, with one addition: each person is an SMPL body, estimated from the clip and refined by the agent, with hair and clothes skinned to it. That gives every person an explicit skeleton, so motion can be edited per person and even per body part. These four clips are longer (121 to 158 frames) and come from films and from multi-person footage.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}hum_lala_recon.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_crowd_recon.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_tango_recon.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">Reconstructions with people: a dance duet, a street with 29 people, and a tango in a restaurant. Source clip, the clothed-SMPL proxy, and our render from the clip's camera.</p>
</div>

**A speed per person.** Each person's whole motion is replayed on their own clock while everyone else keeps the clip's pace. In the duet, the two dancers run at different speeds, here seen from a camera orbiting 45°, so the edit combines motion and camera.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}hum_lala_speed.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_lala_speed_proxy.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_lala_speed180.mp4" autoplay muted loop playsinline></video><figcaption>The same three settings from a 180° orbit, ending behind the dancers.</figcaption></figure>
  </div>
  <p class="cap">One speed per dancer. Final videos, their proxies, and the same edits under a second camera path. Yellow dress, shirt and tie stay on the right person even when the dancers pass each other at a different moment than in the film.</p>
</div>

Out of sync, two bodies can end up in the same place. Before rendering, the harness checks for this and moves them apart: in the duet the dancers are pushed apart smoothly, and in the street two walkers who would collide each step aside by half the distance.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}hum_crowd_speed.mp4" autoplay muted loop playsinline></video><figcaption>Crowded street: two chosen walkers retimed, the others at the clip's pace. A sped-up walker keeps walking past the end of her recorded motion.</figcaption></figure>
    <figure><video src="{{ V }}hum_crowd_speed_proxy.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_tango_speed.mp4" autoplay muted loop playsinline></video><figcaption>Tango: the couple is retimed together, one clock for both, so the embrace stays closed. At 1.5× they finish early and come to rest.</figcaption></figure>
    <figure><video src="{{ V }}hum_paint_speed.mp4" autoplay muted loop playsinline></video><figcaption>Powder fight: two throwers retimed; each one's powder clouds are retimed with them.</figcaption></figure>
  </div>
  <p class="cap">Per-person speed in multi-person scenes.</p>
</div>

**A speed per body part.** With a skeleton, each part can run on its own clock: legs and travel, turning, torso, head, and each arm. A part at speed 0 freezes in its first-frame pose; legs carry the body's position, so planted feet stay planted.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}hum_lala_parts1.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_lala_parts_proxy.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}hum_lala_parts2.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">Per-part edits on the duet, from the clip's own camera. With her arms frozen they stay at her chest while her feet keep tapping; with his turns removed he keeps facing the camera.</p>
</div>

The renders follow the proxies' poses at the frames we checked, and identities survive the dancers overlapping. Two caveats are specific to these scenes. Edited motion is invented motion: in the duet, a dancer at 1.5× runs out of recorded motion two thirds of the way through and plays it backwards, which the later scenes replace with walking on or coming to rest. And faces are plausible but generic where the first frame does not pin them down.

### Visual effects

Each effect below was written by a second agent in 40 to 60 minutes, starting from the reconstructed scene.

<div class="panel">
  <div class="clips two">
    <figure><video src="{{ V }}vfx_car_squish.mp4" autoplay muted loop playsinline></video><figcaption>Squashed by an invisible press</figcaption></figure>
    <figure><video src="{{ V }}vfx_car_burst.mp4" autoplay muted loop playsinline></video><figcaption>Bursts into 73 pieces; the wheels roll on</figcaption></figure>
    <figure><video src="{{ V }}vfx_car_stretch.mp4" autoplay muted loop playsinline></video><figcaption>Stretches like a rubber band</figcaption></figure>
    <figure><video src="{{ V }}vfx_swan_burst.mp4" autoplay muted loop playsinline></video><figcaption>Shatters like a ceramic figurine</figcaption></figure>
    <figure><video src="{{ V }}vfx_horse_squish.mp4" autoplay muted loop playsinline></video><figcaption>Horse and rider squashed, still jumping</figcaption></figure>
    <figure><video src="{{ V }}vfx_bus_burst.mp4" autoplay muted loop playsinline></video><figcaption>Bus bursts apart</figcaption></figure>
    <figure><video src="{{ V }}vfx_cow_burst.mp4" autoplay muted loop playsinline></video><figcaption>Cow shatters</figcaption></figure>
    <figure><video src="{{ V }}vfx_swan_loop.mp4" autoplay muted loop playsinline></video><figcaption>Neck ties itself into a loop</figcaption></figure>
    <figure><video src="{{ V }}vfx_dog_wings.mp4" autoplay muted loop playsinline></video><figcaption>Ears become wings</figcaption></figure>
    <figure><video src="{{ V }}vfx_goat_horns.mp4" autoplay muted loop playsinline></video><figcaption>Horns grow into spirals</figcaption></figure>
    <figure><video src="{{ V }}vfx_boat_foils.mp4" autoplay muted loop playsinline></video><figcaption>Rises on hydrofoils</figcaption></figure>
    <figure><video src="{{ V }}vfx_carr_wheelie.mp4" autoplay muted loop playsinline></video><figcaption>Rounds the bend on its rear wheels</figcaption></figure>
  </div>
  <p class="cap">Effects on the subject itself. The agents chose the last six themselves when asked for "something very different from squashing or shattering".</p>
</div>

Effects can also add an object that was never in the clip. The agent models and animates it, and the subject reacts. We found that the first frame does not need editing for this to work: with the untouched first frame, the diffusion model draws the new object from the proxy and the prompt alone. Painting the object into the first frame with an image model is optional, and gives control over how it looks.

<div class="panel">
  <div class="clips two">
    <figure><video src="{{ V }}vfx_car_hand.mp4" autoplay muted loop playsinline></video><figcaption>A giant hand pinches the car</figcaption></figure>
    <figure><video src="{{ V }}vfx_car_palm.mp4" autoplay muted loop playsinline></video><figcaption>A giant palm presses it flat</figcaption></figure>
    <figure><video src="{{ V }}vfx_car_ufo.mp4" autoplay muted loop playsinline></video><figcaption>A flying saucer abducts it</figcaption></figure>
    <figure><video src="{{ V }}vfx_carr_robot.mp4" autoplay muted loop playsinline></video><figcaption>The car unfolds into a robot, runs, and folds back</figcaption></figure>
    <figure><video src="{{ V }}vfx_swan_fish.mp4" autoplay muted loop playsinline></video><figcaption>A giant fish swallows the swan</figcaption></figure>
  </div>
  <p class="cap">Effects with external objects, and one mechanical transformation.</p>
</div>

The agents build these in quite different ways. The squashed car is a few shape keys driven by weight curves. The break-ups cut the posed mesh into solid pieces and bake a ballistic flight for each. The robot deforms nothing: the car is cut into about sixty rigid panels, and each slides from its place on the car to a docking position on a procedurally animated skeleton.

### Compared with generating the effect directly

A fair question is whether the proxy is needed at all. The base video model can generate from a first frame and a prompt, so we gave unmodified MiniMax-H3 the clip's first frame and the same scene-and-effect description that our renderer received, with no proxy, for all 47 effects we had at the time.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}base_ufo.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_fish.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_robot.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_palm.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_bear_burst.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">Where direct generation does well. Proxy, our render, and the base model from first frame and prompt alone. The base model's clips are 5.2 seconds, its minimum length; ours hold their last frame to match.</p>
</div>

For events the model knows well, direct generation works, and often looks better than ours: its tractor beam is a proper cone of light and its fish is wet and coloured, where ours inherit a faint beam and a grey fish from the proxy. By our count it produced the requested event in all five external-object effects and in essentially all fourteen break-ups.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}base_rhino_squish.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_cow_neck.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_elephant_ears.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_bus_lift.mp4" autoplay muted loop playsinline></video></figure>
    <figure><video src="{{ V }}base_horse_squish.mp4" autoplay muted loop playsinline></video></figure>
  </div>
  <p class="cap">Where it does not. Asked for a squashed rhino, a cow with a giraffe's neck, an elephant flying on its ears, a bus on a scissor lift and a squashed horse and rider, the base model mostly leaves the subject as it was.</p>
</div>

For unusual deformations it mostly does nothing, or turns the request into ordinary behaviour such as an elephant lying down. By our count it produced about 2 of 14 squashes and about 6 of 14 of the agents' own inventions. With the proxy, our render followed the edited geometry in all 47, because the proxy dictates the shape; a few of those results are still hard to read, for example a flattened elephant against pale sand.

| Kind of effect | Base model shows the effect | Our render follows the proxy |
| --- | --- | --- |
| External object or creature | 5 of 5 | 5 of 5 |
| Break-up | about 14 of 14 | 14 of 14 |
| Squash | about 2 of 14 | 14 of 14 |
| Agent's own choice | about 6 of 14 | 14 of 14 |

These counts are a first reading from four frames per clip, not a formal evaluation, and the prompts are long descriptions written from the proxy scenes, which helps the base model. Our reading is that the proxy is not what makes familiar events possible. It is what makes unfamiliar ones happen, and what fixes timing, placement and camera for all of them.

### Adjusting an effect after it is built

The second thing the proxy buys is the ability to change an effect without starting over. Under the parameter contract we built 19 effects on 13 clips. All 19 agents delivered a script with named parameters that passed the check, within their hour.

In each row below, only the first video involved an agent. The others re-run the same script with different numbers.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}param_palm.mp4" autoplay muted loop playsinline></video><figcaption>Giant palm. At half strength the hand stops on a half-crushed cabin; its stop height is computed from the strength.</figcaption></figure>
    <figure><video src="{{ V }}param_bear.mp4" autoplay muted loop playsinline></video><figcaption>Bear shatters. Started later, it breaks further along its walk, from its pose at that frame, with new landings. At half strength it crumbles in place.</figcaption></figure>
    <figure><video src="{{ V }}param_horse.mp4" autoplay muted loop playsinline></video><figcaption>Horse and rider shatter over the fence, after landing, or drop in a heap beside it.</figcaption></figure>
    <figure><video src="{{ V }}param_cow.mp4" autoplay muted loop playsinline></video><figcaption>Giraffe neck. At half length the muzzle cannot reach the trough, so there is no drinking and no ripples; whether it reaches is computed, not scripted.</figcaption></figure>
    <figure><video src="{{ V }}param_fish.mp4" autoplay muted loop playsinline></video><figcaption>Giant fish. At half strength a smaller fish lunges and misses, and the swan swims on.</figcaption></figure>
    <figure><video src="{{ V }}param_wings.mp4" autoplay muted loop playsinline></video><figcaption>Winged horse. Half strength gives small wings and a jump close to the real one.</figcaption></figure>
  </div>
  <p class="cap">Three settings of one script per row: as built, started 8 frames later, and at half strength.</p>
</div>

Half strength is often a different outcome and not a scaled copy: the saucer leaves without the car, the fish misses, the cow does not drink. The effect agent wrote those branches in advance from a one-line description of what "half" should mean, so choosing between them later is still just a parameter.

Agents also add parameters of their own, or ones the brief asks for.

<div class="panel">
  <div class="clips">
    <figure><video src="{{ V }}param_robot.mp4" autoplay muted loop playsinline></video><figcaption>Robot, by strength: the full transformation, arms and head without legs, and a slight rear-up.</figcaption></figure>
    <figure><video src="{{ V }}param_robot_time.mp4" autoplay muted loop playsinline></video><figcaption>Robot, by timing: a shorter run, a faster transformation, and three overrides combined.</figcaption></figure>
    <figure><video src="{{ V }}param_bus.mp4" autoplay muted loop playsinline></video><figcaption>Bus: fewer pieces, and a different random variation of the same burst.</figcaption></figure>
    <figure><video src="{{ V }}param_train.mp4" autoplay muted loop playsinline></video><figcaption>Model train: vehicles bursting one after another, all at once, and with half the throw.</figcaption></figure>
    <figure><video src="{{ V }}param_dog.mp4" autoplay muted loop playsinline></video><figcaption>Dog: two wing beats, one, and half-size wings.</figcaption></figure>
  </div>
  <p class="cap">Other parameters: how long the robot runs, how many pieces the bus breaks into, a random seed, the delay between vehicles, the number of wing beats.</p>
</div>

Not every parameter is a smooth dial. The saucer's strength has a threshold: below 1 the car is always put back. The robot at three-quarter strength is an odd half-car, half-robot that nobody designed. And some parameters, such as how much a squashed rhino wobbles, made little visible difference in the frames we checked.

### What it costs

| Step | Time | How often |
| --- | --- | --- |
| Reconstruction agent | about 60 min | once per clip |
| Effect agent | 25 to 57 min | once per effect |
| Re-running an effect script with new parameters | seconds to 2 min | per setting |
| Grey proxy render | 0.5 to 3 min for most scenes | per edit |
| Diffusion render | about 8 min | per edit |

Camera and time edits need no agent after reconstruction, so they cost one proxy render and one diffusion render. From the agents' timestamped logs, 85 to 95 per cent of an agent's hour is the model reasoning and writing code; waiting on Blender is a few minutes.

## Limitations

- **It is slow to start.** An hour of agent time before the first edit is far more than one diffusion pass with a specialised model. The cost is paid once per clip and once per effect, and later edits take minutes, but a single one-off edit is cheaper elsewhere.
- **The look is limited by the proxy and the renderer.** The grey proxy discards materials, so the saucer's translucent beam arrives as thin lines and renders as a faint glow, and the fish stays grey. Direct generation looks better on effects it can do.
- **Reconstruction is approximate.** Silhouette overlap is 0.67 to 0.91, and the render follows the proxy's errors. Subjects close in colour to their background, such as a flattened rhino beside a log, become hard to read.
- **Consistency covers geometry, not appearance.** Regions the first frame does not show are imagined again in every render. We have not measured how consistent they are across rounds of editing.
- **Parameters are only as good as the script.** The check proves that a parameter does something at four settings. It does not verify contacts or framing at other values, some ranges are narrow on short clips, and prompt text exists only for the settings the agent wrote it for.
- **The evidence is qualitative.** One agent run and one seed per result, a small set of short clips, and counts made by eye.
{: .limits}

## Conclusion

Splitting the work between an agent and a diffusion model gives a video editor with no edit-specific training. The agent supplies what it is good at, an explicit scene in code that can be changed precisely. The diffusion model supplies what it is good at, appearance. Camera, time and visual effects become edits to one scene, they agree with each other because they share it, and an effect, once built, is a program that can be re-run with different numbers. Direct generation is faster and, for familiar events, prettier. The proxy earns its cost when the edit is unusual, when it has to land at a particular moment and place, or when it will need adjusting again.

<script>
  (function () {
    var vids = document.querySelectorAll('.prose video');
    if (!('IntersectionObserver' in window)) return;
    var io = new IntersectionObserver(function (entries) {
      entries.forEach(function (e) {
        if (e.isIntersecting) { var p = e.target.play(); if (p && p.catch) p.catch(function () {}); }
        else e.target.pause();
      });
    }, { rootMargin: '200px' });
    vids.forEach(function (v) { v.pause(); io.observe(v); });
  })();
</script>
