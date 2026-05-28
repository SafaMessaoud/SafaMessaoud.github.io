---
title: "RoBee"
permalink: /projects/robee/
excerpt: "A community-driven platform for collecting, merging, and sharing career roadmaps — Google Maps for career journeys."
image: /images/projects/Robee.png
status: "Ongoing"
order: 4
---

<style>
/* Before/after slider for the origin-story image */
.ba-slider {
  position: relative;
  width: 100%;
  margin: 0.5rem 0 0.6rem 0;
  overflow: hidden;
  border-radius: 6px;
  box-shadow: 0 2px 14px rgba(0,0,0,0.08);
  user-select: none;
  touch-action: none;
}
.ba-slider img {
  display: block;
  width: 100%;
  height: auto;
}
.ba-slider .ba-after-wrap {
  position: absolute;
  inset: 0;
  width: 50%;
  overflow: hidden;
  pointer-events: none;
}
.ba-slider .ba-after-wrap img {
  width: 200%;
  max-width: none;
  height: 100%;
  object-fit: cover;
  object-position: left center;
}
.ba-slider .ba-handle {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 50%;
  width: 2px;
  background: #ffffff;
  box-shadow: 0 0 0 1px rgba(0,0,0,0.18);
  cursor: ew-resize;
  transform: translateX(-50%);
}
.ba-slider .ba-handle::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #ffffff;
  box-shadow: 0 1px 6px rgba(0,0,0,0.22);
  transform: translate(-50%, -50%);
}
.ba-slider .ba-handle::before {
  content: "\2194";
  position: absolute;
  top: 50%;
  left: 50%;
  z-index: 2;
  font-size: 16px;
  color: #444;
  transform: translate(-50%, -50%);
  pointer-events: none;
}
.ba-slider .ba-label {
  position: absolute;
  top: 0.6rem;
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: #fff;
  background: rgba(0,0,0,0.55);
  padding: 0.15rem 0.55rem;
  border-radius: 999px;
  pointer-events: none;
}
.ba-slider .ba-label--before { left: 0.6rem; }
.ba-slider .ba-label--after  { right: 0.6rem; }
.robee-hero figcaption {
  margin-top: 0.4rem;
  font-size: 0.85rem;
  font-style: italic;
  color: var(--global-text-color-light, #777);
  text-align: center;
}
.robee-cta {
  display: inline-block;
  margin-top: 1.25rem;
  padding: 0.55rem 1.15rem;
  background: #6b3df0;
  color: #fff !important;
  border-radius: 6px;
  font-weight: 600;
  text-decoration: none !important;
  letter-spacing: 0.02em;
  transition: background .15s ease, transform .15s ease;
}
.robee-cta:hover {
  background: #552bcc;
  transform: translateY(-1px);
}
.robee-cta::after { content: " \2192"; }
</style>

<figure class="robee-hero">
  <div class="ba-slider" id="robee-ba">
    <!-- Bottom layer: the FULL colour version -->
    <img src="{{ '/images/projects/robee-story-colored.png' | relative_url }}" alt="Coloured illustration of a PhD workspace scene — student at a desk, labmate hanging a chameleon poster, bookshelf labelled DAC, DSN, Xilinx, TA Stuff.">
    <!-- Top layer: the ORIGINAL pencil sketch, clipped to the left side -->
    <div class="ba-after-wrap" id="robee-ba-wrap">
      <img src="{{ '/images/projects/robee-story-original.png' | relative_url }}" alt="Original pencil sketch of the same workspace scene.">
    </div>
    <span class="ba-label ba-label--before">Original</span>
    <span class="ba-label ba-label--after">Coloured</span>
    <div class="ba-handle" id="robee-ba-handle"></div>
  </div>
  <figcaption>Drag the handle to compare the original pencil sketch with the coloured version (by Gemini) &mdash; a scene from my early PhD days, Urbana-Champaign, 2017.</figcaption>
</figure>

<script>
(function() {
  var slider = document.getElementById('robee-ba');
  if (!slider) return;
  var wrap   = document.getElementById('robee-ba-wrap');
  var handle = document.getElementById('robee-ba-handle');
  var dragging = false;

  function setPos(clientX) {
    var rect = slider.getBoundingClientRect();
    var x = clientX - rect.left;
    var pct = Math.max(0, Math.min(100, (x / rect.width) * 100));
    wrap.style.width = pct + '%';
    handle.style.left = pct + '%';
    // Counter-scale the inner image so it never appears squished
    var img = wrap.querySelector('img');
    if (img && pct > 0) {
      img.style.width = (100 / (pct / 100)) + '%';
    }
  }

  function down(e) { dragging = true; move(e); e.preventDefault(); }
  function up()    { dragging = false; }
  function move(e) {
    if (!dragging) return;
    var cx = e.touches ? e.touches[0].clientX : e.clientX;
    setPos(cx);
  }

  slider.addEventListener('mousedown', down);
  window.addEventListener('mousemove', move);
  window.addEventListener('mouseup', up);
  slider.addEventListener('touchstart', down, { passive: false });
  window.addEventListener('touchmove', move, { passive: false });
  window.addEventListener('touchend', up);
})();
</script>

How RoBee came to be
======

The drawing above is from the early days of my PhD. My labmate Homa was taping a poster of a **chameleon** above her desk (the size is exaggerated in my painting). She told me it reminded her of *adaptability* — that the hardest part of a PhD is understanding the system, accepting what you can't change, and adapting to it.

That conversation stuck with me. I realized that a big chunk of the difficulty in a PhD — and well beyond it — comes from not having a clear picture of what lies ahead. Without that picture, we rely on assumptions, and when reality unfolds differently, it can be harder to absorb because we weren't prepared for it. The problem is especially acute for **under-represented communities**, who often don't have easy access to mentors who have walked the road before them.

**RoBee** — short for **Ro**admap **B**uilding **e**xchang**e** — is an attempt to address that challenge. It's a more **scalable approach to mentorship** in which a community shares **roadmaps**: sequences of milestones toward a goal. Individual roadmaps can be merged into **graphs that visualize trajectories**, letting anyone see *all the possible paths* toward a goal, similar to how Google Maps shows you alternate routes between two points. Users can still hold one-on-one mentorship sessions on top of these roadmaps when they need a more personal conversation — RoBee just makes sure no one is starting from a blank map.

What RoBee does
======

- **Collect** — contributors document the milestones, decisions, and detours that shaped their journey.
- **Merge** — similar roadmaps are aligned and combined into community-built graphs, surfacing the full landscape of routes toward each goal.
- **Share & explore** — anyone can browse, follow, or remix a roadmap relevant to where they're going.
- **Mentor on top** — community members can hold mentorship sessions anchored to a shared roadmap, making advice concrete and goal-specific.

The platform is being built as part of the *Mentorship Platform* project funded by the **QRDI Social Innovation Fund** (8th cycle).

<a class="robee-cta" href="#coming-soon">Launch RoBee</a>
