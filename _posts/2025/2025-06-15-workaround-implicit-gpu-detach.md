---
layout: post
title:  Workaround for occasional implicit GPU detachment in Docker
date:   2025-06-15
tags:   ubuntu docker ollama llm gpu nvidia
---
## 課題

[Ubuntu
24.04 LTS](https://canonical.com/blog/canonical-releases-ubuntu-24-04-noble-numbat)
のディストリビュートされているdocker.ioに対して、[Nvidia docker
container](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/index.html)
でNvidia GPUを貼り付けたDockerコンテナを長期ランニングしていると、いつ
の間にかGPUが外れていることがある。

この事象が発生していると、Dockerコンテナのサービスが使用できなかったり、
Dockerコンテナのサービス使用時のGPUの代わりにCPU負荷が急上昇したりする。
そして、Dockerのログには、GPUが利用できないメッセージが出現していたり
する。

例えば、[ollama/ollama](https://hub.docker.com/r/ollama/ollama)の場合、
この現象が発現すると、以下に示すように`no CUDA-capable device`を含んだ
エラーメッセージが出力される。
![no CUDA-capable device](/assets/images/2025/0614/no-CUDA-capeble.png)

## 回避方法

この問題は[NOTICE: Containers losing access to GPUs with error:
"Failed to initialize NVML: Unknown Error"
#48](https://github.com/NVIDIA/nvidia-container-toolkit/issues/48)等で広く
知られている。そして、その事象は、`systemctl daemon-reload`の実行によっ
て簡単に誘発されることが記されている。また、同記事には「5. Workarounds」
で3つの回避方法が提案されている。

1. nvidia-ctkの使用
2. 明示的なsystemd cgroup管理の無効化
3. dockerのダウングレード

今回は、この3つの方法の中、一番容易と思われる2で事態が解消されることを
確認した。

以下に事象の発生の確認と回避方法の検証動画を示す。検証には、Dockerコン
テナ内での`torch.cuda.is_available()`によるGPUの使用可否で確認した。

<video autoplay loop width=640px controls muted>
    <source src="/assets/images/2025/0614/workaround_for_implicit_gpu_detach.webm">
</video>

## 注意

今回の回避策の検証では、DockerのCgroup Driverをsystemdからcgroupfsに変更する
ことで実現している。つまり、Docker全体に影響することを意味する。
このため、製品やサービスへの適用する場合には、十分な見極めが必要と思われる。

## 参考
今回の回避策(systemd → cgroupfs)について、ChatGPTに影響を問合せた結果、
以下の回答となった(回答内容の正確性は未確認)。

> teach me the impact of changing the cgroup driver from `systemd` to `cgroupfs`.
>
> ### ChatGPT:
>
> Changing the cgroup driver in container runtimes like Docker or
> containerd from `systemd` to `cgroupfs` can significantly affect how your
> containers interact with system resources. Here's a breakdown to help
> you understand the impact of this change:
>
> -------------------------------------------------------------------------------
>
> #### 🔧 What is a Cgroup Driver?
>
> Cgroup (control group) drivers are used by container runtimes to
> manage and limit resources (CPU, memory, I/O) for containers using
> Linux cgroups.
>
> - `systemd` driver: Integrates with the `systemd` init system to manage cgroups.
> - `cgroupfs` driver: Uses the legacy cgroup filesystem interface directly, independently of `systemd`.
>
> -------------------------------------------------------------------------------
>
> #### 🧠 Conceptual Difference
>
> | Feature          | `systemd` Driver                                     | `cgroupfs` Driver                          |
> |------------------|----------------------------------------------------|------------------------------------------|
> | Integration      | Tight integration with `systemd`                     | Independent of `systemd`                   |
> | Cgroup hierarchy | Unified and consistent                             | May conflict with `systemd`’s own cgroups |
> | OS compatibility | Best on `systemd`-based systems (e.g., Ubuntu, RHEL) | More universal but less consistent       |
>
> -------------------------------------------------------------------------------
>
> #### ⚠️ Impact of Switching to `cgroupfs`
>
> ##### ✅ Pros
>
> - Compatibility with non-systemd systems: Useful on systems without `systemd` (like Alpine Linux).
> - Simplicity: Sometimes easier to debug since it doesn’t interweave with `systemd`’s own cgroups.
>
> ##### ❌ Cons
>
> 1. Potential conflicts:
>     - `systemd` and `cgroupfs` may manage the same hierarchy, causing conflicts in ownership or nested cgroups.
>     - You may see errors like:
>
> 		```sql
> 		failed to create shim task: OCI runtime create failed: ... cgroup conflict: ...: unknown
>         ```
> 2. Monitoring and tooling inconsistency:
> - Tools that rely on `systemd` cgroup hierarchy (e.g., `systemd-cgtop`, `journalctl`) may not accurately report container metrics.
> 3. Cluster management issues (e.g., Kubernetes):
>     - Kubernetes expects node cgroup drivers to match between the kubelet and container runtime. Mismatched drivers (e.g., kubelet using `systemd`, but Docker using `cgroupfs`) can cause:
>         - Pod startup failures
> 		- Resource management errors
> 		- Node tainting or eviction
> 4. Security & isolation nuances:
>     - `systemd` may better isolate services by placing each in its own slice and controlling them more granularly.
>
> -------------------------------------------------------------------------------
>
> #### 🧩 Summary
> Changing the cgroup driver from `systemd` to `cgroupfs`:
> - Is not just a cosmetic or config change — it affects how containers are integrated with your OS.
> - May lead to issues if your system or Kubernetes setup assumes a specific driver.
> - Should be done consistently across your container runtime and orchestration components.
