---
title: Metastability in open quantum systems
summary: Understanding the long time properties of quantum dynamics.
tags:
- Open quantum systems
- Nonequilibrium physics
- Large deviation theory
date: "2020-06-22T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Flow of spin states to the metastable manifold in a glassy system.
  focal_point: Smart

links:
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

In this project, leading on from [foundational work](https://link.aps.org/doi/10.1103/PhysRevLett.116.240404) [(ArXiv)](https://arxiv.org/abs/1512.05801) on metastability in open quantum systems, we study the metastability present in a variety of models, further developing this theory in the process.

Beginning with a simple [adaptation of the transverse field Ising model]({{< ref "/publication/phys-rev-e-94-052132/index.md" >}}), we examined how the long time dynamics reduces to a simple two-state jump process, between phases corresponding to paramagnetic and ferromagnetic statistics. Along with its presence in the vicinity of phase transitions, metastability is expected in systems with constrained dynamics, such as glassy systems: this lead to our next study, focused on a [dissipative quantum East model](https://link.aps.org/doi/10.1103/PhysRevLett.109.020403) [(ArXiv)](https://arxiv.org/abs/1203.6585). While the results for this model are still in preparation, in the process we had to first develop a numerical approach in order to study systems exhibiting metastability with more than two phases. This development was accompanied by an extensive set of mathematical results for [classical metastability]({{< ref "/publication/macieszczak-2020-theory/index.md" >}}) in open quantum systems, which is now available on [ArXiv](https://arxiv.org/abs/2006.01227) along with the current numerical setup.

In the meantime, an investigation into the affect of resets on metastable processes lead to far more [general results]({{< ref "/publication/phys-rev-e-98-022129/index.md" >}}) about the spectra of reset processes, both classical and quantum. A corollary of these general results was their implications for systems exhibiting metastability, with varying effects depending on the rate and target state of the resets.