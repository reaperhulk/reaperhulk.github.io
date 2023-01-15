---
layout: post
title: Ephemeral GHA Apple Silicon Runners with CiderMill
---

In the <a href="https://github.com/pyca/cryptography">pyca/cryptography</a> project we ostensibly write Python (and Rust), but in reality we are CI engineers doomed to spend 90% of all our development time <a href="https://frinkiac.com/meme/S08E25/1251900/m/IFlPVVIgRFVUWSBJUyBDTEVBUi0tIFRPCiBCVUlMRCBBTkQgTUFJTlRBSU4gVEhPU0UKIFJPQk9UUy4=">building and maintaining</a> our <a href="https://github.com/pyca/cryptography/tree/965af65ae7bcc23ff1b3d123b95c0b7b29d2795b/.circleci">annoyingly</a>[^12] <a href="https://github.com/pyca/cryptography/tree/965af65ae7bcc23ff1b3d123b95c0b7b29d2795b/.github">large</a> CI[^11]. As part of that unwanted work we possess a self-hosted runner for macOS arm64 builds[^7].

Self-hosted runners have one big drawback[^1] -- they lack build isolation. In the typical security model of GitHub Actions every job runs in an isolated environment. At the end of a run that environment is destroyed[^2] and future runs are not exposed to any products of a previous run. In a self-hosted runner, **this is not true**. Instead, each invocation of the runner is running on the same state-filled system that you built, with whatever mutations the last job performed still present. In some cases (e.g. closed source development using GHA) this represents an annoyance (you can't treat all your GHA runners the same!), but can be worked around. However, in the open source world[^3] this represents an unacceptable security problem. CI systems run arbitrary code[^4] and trivial persistence on a CI runner is clearly undesirable.

For this reason, we have constrained our M1 runner to only pushes to main since we installed it, but, <a href="https://frinkiac.com/meme/S05E16/479261/m/UFJPRkVTU0lPTkFMIEFUSExFVEVTLS0gCkFMV0FZUyBXQU5USU4nIE1PUkUu">like professional athletes</a>, we always want more.

## <a href="https://github.com/reaperhulk/cidermill">CiderMill</a>[^15], ephemeral macOS VM runners for GHA

We created <a href="https://github.com/reaperhulk/cidermill">CiderMill</a> to get the ephemeral macOS arm64[^14] bliss we craved but <a href="https://github.com/github/roadmap/issues/528">didn't want to wait for</a>. CiderMill uses <a href="https://github.com/cirruslabs/tart">tart</a> and <a href="https://github.blog/changelog/2021-09-20-github-actions-ephemeral-self-hosted-runners-new-webhooks-for-auto-scaling/">GHA's ephemeral runner support</a> to provide build isolation while integrating extremely well with the GitHub Actions ecosystem. Simply put it does the following:
* Boots one or more[^10] fresh macOS VMs set up via your own customer Packer scripts using tart (which, in turn, leverages Apple's Virtualization.framework).
* Installs and registers an ephemeral action runner within the VM.
* Waits until the runner exits (upon completion of a job) and then does it all again.

In pyca/cryptography's case thanks to <a href="https://github.com/reaperhulk/cidermill">CiderMill</a> we can relax our constraints and <a href="https://github.com/pyca/cryptography/pull/8066/files">integrate our macOS arm64 job into our primary CI</a>! Less lines of nested bash and YAML plus pre-merge testing on an important architecture, seems good.

We run this on a single 16GB M1 Mini, but projects with more resources can simply run more instances of <a href="https://github.com/reaperhulk/cidermill">CiderMill</a> as needed[^13].

Head on over to <a href="https://github.com/reaperhulk/cidermill">CiderMill</a> to check out the documentation[^6], try it out, and see if it fits your needs.




[^1]: Beyond the **really** big one where you have to operate your own infrastructure.
[^2]: Modulo whatever caching you may have chosen to do via actions/cache, etc.
[^3]: Specifically, open source projects that allow anyone to submit PRs.
[^4]: First-time contributor approval workflows and other things exist to mitigate a variety of CI abuse (e.g. crypto miner), but self-hosted runners have significantly more challenges that are not fully addressable via these mechanisms.
[^6]: This project was built for our own use so there are some documented missing features (e.g. no PATs, only GH Apps). Will we add them? Maybe. Could you add them? Yes!
[^7]: An M1 kindly donated by MacStadium's <a href="https://www.macstadium.com/opensource">Open Source Program</a>.
[^9]: Our CI system is <a href="https://github.com/pyca/cryptography/tree/965af65ae7bcc23ff1b3d123b95c0b7b29d2795b/.circleci">relatively</a> <a href="https://github.com/pyca/cryptography/tree/965af65ae7bcc23ff1b3d123b95c0b7b29d2795b/.github">large</a>.
[^10]: Apple's godawful rules about macOS virtualization <a href="https://eclecticlight.co/2022/08/04/virtualisation-on-apple-silicon-macs-8-how-apple-limits-vms/">limit us</a> to 2 VMs.
[^11]: We run over 70 separate jobs per PR, and this is despite aggressively limiting our combinatorial behavior.
[^12]: There are challenges with a matrix this large, mostly around CI cycle time, tail latency issues, and ephemeral problems with networking or other parts of the underlying infrastructure.
[^13]: Just remember, there's no advantage to running this on an M1 Pro/Max/Ultra machine unless you need more RAM since you're constrained to 2 VMs[^10] per physical machine.
[^14]: Apple Silicon, arm64, AArch64, M1, M2...sigh, too many names.
[^15]: Look at that marketing prowess, a big link to click!
