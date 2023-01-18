# 基于CPU TEE的SPDM通信协议实现

## 背景

机密计算技术基于硬件可执行环境，为用户在使用（计算）过程中的敏感数据提供了机密性和完整性的保护。用户作为敏感数据的拥有者，能够通过机密计算技术提供的远程证明机制使自己相信自己的工作负载确实运行在真实的机密计算CPU TEE里，即机密计算CPU TEE具有可自证性。因此，只要用户只要相信CPU制造商提供的机密计算CPU TEE具有声称的内存加密和隔离等安全能力，相信这种硬件能力能够保证运行时内存安全，就能够相信平台拥有者确实使用了真实的机密计算技术来保护用户应用和数据的机密性和完整性。

除了CPU TEE外，多个标准化组织正在协同配合，定义Device TEE的相关标准，并提供了参考代码实现。机密计算领域的下一步发展方向将是基于CPU TEE和Device TEE共同打造的节点级机密互联架构，这就需要Device TEE与CPU TEE之间能够通过安全可信的通信协议完成链路级和消息级的控制交互和敏感数据传输。

## 拟解决技术问题

为了在执行远程证明过程的时候能够实现不同TEE硬件平台的互通性，需要在以下两个层面实现标准化：
- 机密计算远程证明架构
- 远程证明参与方之间的通信协议

其中前者已经由IETF下的RATS工作组正在编写RATS architecture规范，后者目前还缺少相关标准。根据RATS architecture规范的定义，远程证明过程由attester和verifier这两个最为重要的角色通过远程证明协议协同配合来完成。在Device TEE架构中，同样存在上述角色，并且它们之间的消息级协议Security Protocol and Data Model（SPDM）已经完成。

根据SPDM规范的定义，现代计算平台中的平台管理子系统包括了一组组件，并希望在与组件进行安全通信之前，与组件先建立信任。组件的类型包括但不限于外部设备，也包括SoC内嵌设备、BMC等，甚至包括CPU TEE。

因此，通过在CPU TEE远程证明场景中复用SPDM规范定义的标准化消息协议，SPDM消息协议能够提供了一种建立信任的CPU TEE身份认证机制，该机制使用经过验证的加密方法来保护CPU TEE的身份认证过程。此外，作为在两个端点之间建立信任的一部分，SPDM消息协议还允许创建会话以在端点之间交换安全消息。

最后，SPDM消息协议需要与底层的传输层协议和规范进行绑定，比如目前SPDM消息协议已经绑定了MCTP用于平台管理子系统，绑定了PCIe用于设备数据传输。在CPU TEE远程证明场景中，SPDM消息协议需要绑定的底层传输协议可以是(D)TLS 1.3、HTTPS、gRPC over TLS等任何基于TCP/IP协议栈的传输协议。

在具体实现上，除了参考SPDM规范外，需要在verifier侧基于libspdm库实现SPDM消息协议中的请求者角色的功能，在attester侧基于spdm-emu在Intel SGX或AMD SEV等CPU TEE平台上实现SPDM消息协议中的响应者角色的功能。Attester和Verifier之间支持的传输层协议可以是(D)TLS 1.3、HTTPS、gRPC over TLS等协议中的一种。

## 参考资料

- https://zhuanlan.zhihu.com/p/434718252

- https://zhuanlan.zhihu.com/p/435752580

- https://download.01.org/intel-sgx/Releases/

- https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html

- https://developer.amd.com/sev/
 
- Security Protocols and Data Models Working Group：https://www.dmtf.org/standards/spdm

- （DSP2058）Security Protocol and Data Model (SPDM) Architecture White Paper Version 1.2.0：https://www.dmtf.org/dsp/DSP2058

- （DSP0274）Security Protocol and Data Model (SPDM) Specification Version 1.2.1：https://www.dmtf.org/dsp/DSP0274

- （DSP0277）Secured Messages using SPDM Specification Version 1.1.0：https://www.dmtf.org/dsp/DSP0277

- IETF RATS architecture：https://datatracker.ietf.org/doc/html/draft-ietf-rats-architecture-22

- libspdm：https://github.com/DMTF/libspdm

- spdm-emu：https://github.com/DMTF/spdm-emu

- spdm-dump: https://github.com/DMTF/spdm-dump

- SPDM-Responder-Validator: https://github.com/DMTF/SPDM-Responder-Validator
