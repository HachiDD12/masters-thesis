# Learning from Human Videos for Robotic Manipulation

This repository contains the full source code for [Aditya Kannan's](adityak77.github.io) Master's Thesis [document](http://reports-archive.adm.cs.cmu.edu/anon/2023/abstracts/23-124.html).


## Publications behind this thesis

Some of the content here is behind these publications:

<table class="table table-hover">
<tr>
<td>
<strong>DEFT: Dexterous Fine-Tuning for Real-World Hand Policies</strong><br />
<strong>Aditya Kannan*</strong>, Kenneth Shaw*, Pragna Mannam, Shikhar Bahl, Deepak Pathak<br />
CoRL 2023<br />
[<a href="https://dexterous-finetuning.github.io/" target="_blank">web</a>]
[<a href="https://arxiv.org/abs/2310.19797" target="_blank">arXiv</a>]
[<a href="https://arxiv.org/pdf/2310.19797.pdf" target="_blank">pdf</a>]
[<a href="https://github.com/adityak77/deft-data" target="_blank">data</a>] <br />
</td>
</tr>
</table>

---

The experimental source code and data produced for this
thesis are freely available as open source software and
are available in the following repositories.

+ [adityak77/**deft-data**](https://github.com/adityak77/deft-data) &mdash;
  Training dataset for "DEFT: Dexterous Fine-Tuning for Real-World Hand Policies", an approach that fine-tunes an affordance prior from human videos for real-world kitchen tasks.
+ [adityak77/**rewards-from-human-videos**](https://github.com/adityak77/rewards-from-human-videos) &mdash;
  Our method learns an agent- and domain-agnostic representation that can be applied as a zero-shot reward function for robotic control.
+ [adityak77/**inpainting**](https://github.com/adityak77/inpainting) &mdash;
  Various segmentation and video inpainting approaches with the objective of use towards reward learning. Accompanies code for Rewards from Human Videos.

------


+ This repository is based on
  [Ellis Brown](https://ellisbrown.github.io/)'s [thesis repo](https://github.com/ellisbrown/cmu-masters-thesis),
  which started from [Brandon Amos](http://bamos.github.io)'s [thesis repo](https://github.com/bamos/thesis),
  which was based on [Cyrus Omar's thesis code](https://github.com/cyrus-/thesis),
  which, in turn, was based on a CMU thesis template
  by [David Koes](http://bits.csb.pitt.edu/)
  and others before.

------



The BibTeX for this document is:

```bibtex
@mastersthesis{kannan2023learning,
  author       = {Aditya Kannan},
  title        = {Learning from Human Videos for Robotic Manipulation},
  school       = {Carnegie Mellon University},
  year         = 2023,
  month        = July,
}
```
