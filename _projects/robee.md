---
title: "RoBee: A Roadmap Building Platform for Career Journeys"
permalink: /projects/robee/
excerpt: "A community-driven platform for collecting, merging, and sharing career roadmaps — Google Maps for career journeys."
image: /images/projects/Robee.png
status: "Ongoing"
order: 4
---

{% raw %}
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
  -webkit-user-select: none;
  touch-action: none;
  cursor: ew-resize;
}
.ba-slider img {
  display: block;
  width: 100%;
  height: auto;
  pointer-events: none;
  -webkit-user-drag: none;
}
.ba-slider .ba-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  clip-path: inset(0 50% 0 0);
  -webkit-clip-path: inset(0 50% 0 0);
}
.ba-slider .ba-handle {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 50%;
  width: 2px;
  background: #ffffff;
  box-shadow: 0 0 0 1px rgba(0,0,0,0.18);
  transform: translateX(-50%);
  pointer-events: none;
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
  z-index: 3;
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
    <!-- Bottom layer: COLOURED version (always fully visible) -->
    <img src="/images/projects/robee-story-colored.jpg" alt="Coloured illustration of a PhD workspace scene — student at a desk, labmate hanging a chameleon poster.">
    <!-- Top layer: ORIGINAL pencil sketch, clipped via clip-path -->
    <img class="ba-overlay" id="robee-ba-overlay" src="/images/projects/robee-story-original.jpg" alt="Original pencil sketch of the same workspace scene.">
    <span class="ba-label ba-label--before">Original</span>
    <span class="ba-label ba-label--after">Coloured</span>
    <div class="ba-handle" id="robee-ba-handle"></div>
  </div>
  <figcaption>Drag the handle to compare the original pencil sketch with the coloured version (by Gemini) &mdash; a scene from my early PhD days, Urbana-Champaign, 2014.</figcaption>
</figure>

<script>
(function () {
  function init() {
    var slider  = document.getElementById('robee-ba');
    if (!slider) return;
    var overlay = document.getElementById('robee-ba-overlay');
    var handle  = document.getElementById('robee-ba-handle');
    var dragging = false;

    function setPct(pct) {
      pct = Math.max(0, Math.min(100, pct));
      var inset = 'inset(0 ' + (100 - pct) + '% 0 0)';
      overlay.style.clipPath = inset;
      overlay.style.webkitClipPath = inset;
      handle.style.left = pct + '%';
    }

    function pctFromEvent(e) {
      var rect = slider.getBoundingClientRect();
      var cx = (e.touches && e.touches[0]) ? e.touches[0].clientX : e.clientX;
      return ((cx - rect.left) / rect.width) * 100;
    }

    function onDown(e) {
      dragging = true;
      setPct(pctFromEvent(e));
      if (e.cancelable) e.preventDefault();
    }
    function onMove(e) {
      if (!dragging) return;
      setPct(pctFromEvent(e));
      if (e.cancelable) e.preventDefault();
    }
    function onUp() { dragging = false; }

    slider.addEventListener('mousedown', onDown);
    document.addEventListener('mousemove', onMove);
    document.addEventListener('mouseup', onUp);
    slider.addEventListener('touchstart', onDown, { passive: false });
    document.addEventListener('touchmove', onMove, { passive: false });
    document.addEventListener('touchend', onUp);
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
})();
</script>
{% endraw %}

How RoBee came to be
======

This painting recreates a scene from my early PhD days.

My labmate, Homa, was hanging a small chameleon picture above her desk (the size is admittedly exaggerated in the painting). She told me it reminded her of adaptability—one of the most important skills in a PhD. Success, she said, often comes from learning to understand the system, accept what you cannot change, and adapt to uncertainty.

That conversation stayed with me as the years unfolded.

Over time, I realized that one of the heaviest burdens of a PhD—and many ambitious career journeys beyond it—is navigating the fog of the unknown. We build expectations from assumptions, and when reality unfolds differently, adapting becomes difficult simply because we were never given a realistic view of the road ahead.

I also realized that this challenge disproportionately affects underrepresented communities, where access to mentors, networks, and informal guidance is often limited. Much of the knowledge that determines success is never written down; it is passed through casual conversations and friendships.

RoBee emerged from a desire to democratize that undocumented knowledge.

When navigating uncertainty, I didn’t just need a single mentor's advice. I needed a map. I wanted to see the trajectories others had taken toward their goals—the milestones they reached, the detours they encountered, and the decisions they made along the way.

The platform is built to do exactly that. RoBee allows individuals to share their journeys as structured roadmaps. These individual roadmaps can then be combined into larger maps that reveal multiple paths toward the same destination. By making collective experience visible, RoBee helps others explore alternatives, set realistic expectations, and make more informed decisions about their futures.

Turning this vision into reality has been a deeply collaborative journey. My close friend Fadoua, who shared a similar vision, joined me as a co-founder. Dr. Dena Al Thani provided the critical spark, encouraging us to pursue the idea through HBKU and introducing us to Younss, who completed our founding team. Today, RoBee is being developed through the Mentorship Platform project, funded by the Qatar Research, Development, and Innovation (QRDI) Council Social Innovation Fund (8th Cycle).

Under the hood, RoBee is a platform designed to collect and model causal graphs. From a research perspective, we are currently working on computational methods to efficiently collect and merge these roadmaps.



<a class="robee-cta" href="#coming-soon">RoBee launch page</a>
