
# private-join-and-compute-gm

**private-join-and-compute-gm** 是基于 [Google/private-join-and-compute](https://github.com/google/private-join-and-compute)
commit [`e81563e`](https://github.com/google/private-join-and-compute/commit/e81563e0e3caa0f6178e8668be8a4434d2cef352)
的国密扩展版本，增加了对 **SM2 / SM3 / SM4** 国密算法的支持。

该项目旨在为中国密码算法体系下的 **私有集合交集 (PSI)**、**安全多方计算 (MPC)** 和 **隐私保护分析 (SCQL)** 等应用场景提供兼容实现。

---

## 🔍 项目简介

Google 的 *Private Join and Compute (PJC)* 是一种隐私保护协议，允许多个参与方在不泄露原始数据的情况下，对各自数据集的交集执行计算。
该扩展版在原有的 **椭圆曲线加密 (ECC)** 框架基础上，增加了 **国密 SM 系列算法支持**，从而适配中国国家商用密码标准。

---

## 🧩 新增内容

| 模块                          | 功能        | 说明                         |
| --------------------------- | --------- | -------------------------- |
| ✅ `SM2`                     | 椭圆曲线公钥算法  | 支持密钥生成、加解密、签名与验签           |
| ✅ `SM3`                     | 哈希算法      | 用于哈希到曲线、消息摘要计算             |
| ✅ `Hash-to-Curve (RFC9380)` | SM2曲线映射支持 | 实现了 `hash_to_curve_sm2` 接口 |
---





## 🧠 技术要点

* 实现 `HashToCurveInternal` 支持 SM3 + SM2 映射（RFC9380 风格 SSWU）
* 在 `ECCommutativeCipher` 层增加 GM 模式分支
* 支持 `NID_sm2`、`EVP_sm3()` 等对象标识符注册
* 兼容 BoringSSL-GM 库

---

## 🧾 版本信息

| 项目      | 版本                                                                                                                                                                           |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 上游项目    | [Google/private-join-and-compute@e81563e0e3caa0f6178e8668be8a4434d2cef352](https://github.com/google/private-join-and-compute/tree/e81563e0e3caa0f6178e8668be8a4434d2cef352) |
| 国密扩展作者  | CHENHE                                                                                                                                                                           |
| 国密支持算法  | SM2 / SM3                                                                                                                                                               |
| 哈希到曲线支持 | RFC 9380（SM2曲线）                                                                                                                                                              |

---

## 📚 参考项目

* [Google Private Join and Compute](https://github.com/google/private-join-and-compute)
* [BoringSSL-GM](https://github.com/HunterChan-SR/BoringSSL-GM)

---

## 📄 License

本项目遵循 **Apache License 2.0**。
本仓库仅作为研究用途的扩展版，不保证生产安全性。

---

# 原README:

# Private Join and Compute

This project contains an implementation of the "Private Join and Compute"
functionality. This functionality allows two users, each holding an input file,
to privately compute the sum of associated values for records that have common
identifiers.

In more detail, suppose a Server has a file containing the following
identifiers:

Identifiers |
----------- |
Sam         |
Ada         |
Ruby        |
Brendan     |

And a Client has a file containing the following identifiers, paired with
associated integer values:

Identifiers | Associated Values
----------- | -----------------
Ruby        | 10
Ada         | 30
Alexander   | 5
Mika        | 35

Then the Private Join and Compute functionality would allow the Client to learn
that the input files had *2* identifiers in common, and that the associated
values summed to *40*. It does this *without* revealing which specific
identifiers were in common (Ada and Ruby in the example above), or revealing
anything additional about the other identifiers in the two parties' data set.

Private Join and Compute is a variant of the well-studied Private Set
Intersection functionality. We sometimes also refer to Private Join and Compute
as Private Intersection-Sum.

## How to run the protocol

In order to run Private Join and Compute, you need to install Bazel, if you
don't have it already.
[Follow the instructions for your platform on the Bazel website.](https://docs.bazel.build/versions/master/install.html)

You also need to install Git, if you don't have it already.
[Follow the instructions for your platform on the Git website.](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Once you've installed Bazel and Git, open a Terminal and clone the Private Join
and Compute repository into a local folder:

```shell
git clone https://github.com/google/private-join-and-compute.git
```

Navigate into the `private-join-and-compute` folder you just created, and build
the Private Join and Compute library and dependencies using Bazel:

```bash
cd private-join-and-compute
bazel build //private_join_and_compute:all
```

(All the following instructions must be run from inside the
private-join-and-compute folder.)

Next, generate some dummy data to run the protocol on:

```shell
bazel-bin/private_join_and_compute/generate_dummy_data --server_data_file=/tmp/dummy_server_data.csv \
--client_data_file=/tmp/dummy_client_data.csv
```

This will create dummy data for the server and client at the specified
locations. You can look at the files in `/tmp/dummy_server_data.csv` and
`/tmp/dummy_client_data.csv` to see the dummy data that was generated. You can
also change the size of the dummy data generated using additional flags. For
example:

```shell
bazel-bin/private_join_and_compute/generate_dummy_data \
--server_data_file=/tmp/dummy_server_data.csv \
--client_data_file=/tmp/dummy_client_data.csv --server_data_size=1000 \
--client_data_size=1000 --intersection_size=200 --max_associated_value=100
```

Once you've generated dummy data, you can start the server as follows:

```shell
bazel-bin/private_join_and_compute/server --server_data_file=/tmp/dummy_server_data.csv
```

The server will load data from the specified file, and wait for a connection
from the client.

Once the server is running, you can start a client to connect to the server.
Create a new terminal and navigate to the private-join-and-compute folder. Once
there, run the following command to start the client:

```shell
bazel-bin/private_join_and_compute/client --client_data_file=/tmp/dummy_client_data.csv
```

The client will connect to the server and execute the steps of the protocol
sequentially. At the end of the protocol, the client will output the
Intersection Size (the number of identifiers in common) and the Intersection Sum
(the sum of associated values). If the protocol was successful, both the server
and client will shut down.

## Caveats

Several caveats should be carefully considered before using Private Join and
Compute.

### Security Model

Our protocol has security against honest-but-curious adversaries. This means
that as long as both participants follow the protocol honestly, neither will
learn more than the size of the intersection and the intersection-sum. However,
if a participant deviates from the protocol, it is possible they could learn
more than the prescribed information. For example, they could learn the specific
identifiers in the intersection. If the underlying data is sensitive, we
recommend performing a careful risk analysis before using Private Join and
Compute, to ensure that neither party has an incentive to deviate from the
protocol. The protocol can also be supplemented with external enforcement such
as code audits to ensure that no party deviates from the protocol.

### Maliciously Chosen Inputs

We note that our protocol does not authenticate that parties use "real" input,
nor does it prevent them from arbitrarily changing their input. We suggest
careful analysis of whether any party has an incentive to lie about their
inputs. This risk can also be mitigated by external enforcement such as code
audits.

### Leakage from the Intersection-Sum.

While the Private Join and Compute functionality is supposed to reveal only the
intersection-size and intersection-sum, it is possible that the intersection-sum
itself could reveal something about which identifiers were in common.

For example, if an identifier has a very unique associated integer values, then
it may be easy to detect if that identifier was in the intersection simply by
looking at the intersection-sum. One way this could happen is if one of the
identifiers has a very large associated value compared to all other identifiers.
In that case, if the intersection-sum is large, one could reasonably infer that
that identifier was in the intersection. To mitigate this, we suggest scrubbing
inputs to remove identifiers with "outlier" values.

Another way that the intersection-sum may leak which identifiers are in the
intersection is if the intersection is too small. This could make it easier to
guess which combination of identifiers could be in the intersection in order to
yield a particular intersection-sum. To mitigate this, one could abort the
protocol if the intersection-size is below a certain threshold, or to add noise
to the output of the protocol.

(Note that these mitigations are not currently implemented in this open-source
library.)

## Disclaimers

This is not an officially supported Google product. The software is provided
as-is without any guarantees or warranties, express or implied.
