# 服务网格应用接入规范

为了您的应用能够接入网格，并体验到所有网格的能力，我们需要对应用做一些规范。
本文档将介绍服务网格的基本原理和结构，以及应用接入服务网格的规范和建议。

当然，如果您的应用在接入网格之后，没有按照预期运行，也可以按照此文档进行排查，确认应用配置是否符合规范。

## 引言

### 服务网格概述

服务网格（Service Mesh）是一种基础设施层，用于处理服务到服务之间的通信。
在一个微服务架构中，服务网格帮助构成其复杂拓扑，以便它可以更好地运行、观察、保护和配置微服务。
服务网格通常用于大规模的企业应用，其中包含大量的微服务。

我们的服务网格产品，基于 Istio 开源项目，提供了一套完整的服务网格解决方案，
包括对多云应用的支持、对多种开发语言的支持、对多种部署方式的支持、对多种应用场景的支持等。

### 规范目的和适用范围

此《服务网格应用接入规范》的目的在于提供一个具体的指南，以帮助开发者和团队理解服务网格的核心概念和工作流程，
并正确地将应用接入服务网格，确保服务稳定、安全、高效运行。

本规范主要适用于有一定微服务基础知识，希望进一步理解和使用服务网格技术的技术人员。
无论你是新接触服务网格，还是已经有一定了解，希望深化对接入过程的理解，本规范都会为你提供详细的指南和实用建议。

## 服务网格基本原理与结构

服务网格是一种用于处理服务到服务之间通信的基础设施层。它负责在微服务架构中的各个服务之间进行可靠、快速和安全的网络请求传输。
服务网格通过提供服务发现、负载均衡、故障恢复和服务访问策略等功能，帮助开发者专注于构建业务逻辑，而不必关心服务间通信的细节。

在服务网格架构中，网络通信被抽象化为服务间的请求和响应。这些请求和响应通过服务网格在服务间进行路由，
并由网格负责处理重试、超时、断路、身份验证等问题。开发者只需要声明他们希望如何处理请求和响应，服务网格则负责实现这些策略。

为了实现这些功能，服务网格通常由两部分组成：数据平面和控制平面。

- 数据平面由一组网络代理组成，它们在应用程序的每个服务实例中作为边车运行，处理进出该实例的网络通信。
- 控制平面管理和配置数据平面的代理，以及提供服务网格的管理功能，如流量管理、策略配置、服务发现、认证、监控和报告等。

因为引入了边车，所以我们需要对接入网格的应用做一些规范，确保应用能够正常运行。

## 服务接入规范与建议

### 接入流程概述

在将您的应用接入服务网格的过程中，需要注意以下几个步骤以确保接入的顺利进行：

- **了解服务网格**：在应用接入之前，开发者需要了解服务网格的基本概念、核心组件以及运作方式。
  通过对服务网格的理解，可以帮助开发者更好地整合应用。
- **评估应用的适配性**：评估您的应用是否适合在服务网格环境下运行，例如是否兼容服务网格的通信协议、是否能在容器化环境下运行等。
  此外，开发者也需要理解服务网格带来的变化，例如流量的路由、负载均衡等都由服务网格控制。
- **应用的修改与优化**：根据服务网格的规范，可能需要对应用进行一些修改，如将健康检查、日志、跟踪等信息暴露出来，
  以便服务网格进行监控和管理。同时，也要考虑如何处理服务网格在应用外部处理的功能，例如重试、超时控制等。
- **接入服务网格**：根据服务网格提供的接口或 SDK，进行代码级别的接入，这通常包括导入相关库，
  初始化服务，设置服务发现、负载均衡策略等。
- **测试与优化**：在应用接入服务网格后，需要进行充分的测试，以确保应用在服务网格环境下的表现和性能。
  此外，根据测试结果，可能需要对应用或服务网格的配置进行进一步的优化。

### 应用运行环境要求

由于在同一个 Pod 实例中引入了边车，所以在接入服务网格时，需要对应用的运行环境做一些要求，具体如下：

| 要求项                        | 是否必须 | 要求值                                                                                                                                                           | 说明                                                                                                                               |
| ----------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 端口监听                      | 是       | 应用不可以监听以下端口：<br><li>15000 - 15010</li><br><li>15020 - 15021</li><br><li>15053</li><br><li>15090</li>                                             | 由于服务网格的数据平面需要监听这些端口，所以应用不可以监听这些端口。                                                               |
| Pod 标签                      | 是       | Pod `app` 和 `version` 标签的值必须与所关联的 `Service` 名称相同，注意不是如 Deploment 自己的标签，而是 Pod（通常位于 `.spec.template.metadata.labels`）的标签。 | 观测、流量泳道等功能都是基于 Pod 的 `app` 和 `version` 标签进行的，所以必须保证标签和值与 `Service` 相同。                     |
| 运行用户 uid                  | 是       | 不能以 uid 为 1337 的用户运行                                                                                                                                    | 由于服务网格的数据平面需要以 uid 为 1337 的用户运行，边车的流量拦截不会处理这个用户的流量，所以应用不能以 uid 为 1337 的用户运行。 |
| HostNetwork                   | 是       | Pod 不能以 HostNetwork 模式运行                                                                                                                                  | HostNetwork 模式不受服务网格支持                                                                                                   |
| DNSPolicy                     | 是       | Pod 的 DNSPolicy 建议为 ClusterFirst，且 ndots 建议设置成 5。                                                                                                    | 由于边车需要与控制面通讯，需要依赖 DNS 解析控制面地址                                                                              |
| 包含 Envoy 进程的应用 base-id | 否       | 如果业务应用运行了 Envoy，需要增加 `--base-id XXX` 参数                                                                                                          | 因为边车使用了 Envoy，如果不加 `--base-id` 参数，两个 Envoy 无法共存                                                               |

### 应用通讯规范

| 要求项       | 是否必须 | 要求值                                                                                                                                                | 说明                                                                                                                                                                                                                                              |
| ------------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 服务访问方式 | 否       | 服务之间访问，需要以服务名的方式进行访问，或者 ClusterIP，不可以直接使用 Pod IP 或者 NodePort。                                                       | 由于服务网格的数据平面需要以服务名的方式策略匹配，所以应用不可以直接使用 Pod IP 或者 NodePort，否则可能造成策略失效或者无法访问。                                                                                                                 |
| 端口协议     | 否       | 正确配置 Service 的端口的协议，且在多集群模式下，确保服务在所有集群的配置保持一致（即同一个 ns 下的同名服务（Service），Service.spec 应当保持一致）。 | 可在 DCE 5.0 服务网格界面（`服务管理`-`服务列表`-`地址信息`）中修改具体端口的协议，或者按照 [Istio 文档](https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection/)进行配置。错误的配置可能引起访问问题或者策略问题。 |

### 对接链路追踪

我们目前使用 [OpenTelemetry](https://opentelemetry.io/) 作为链路追踪的标准，并默认会将链路信息上报到可观测模块，如果您希望使用链路追踪功能，需要在应用中引入 OpenTelemetry 的 SDK，并在应用中配置好链路追踪的参数，具体可以参考 [OpenTelemetry 官方文档](https://opentelemetry.io/docs/)。

或者如果想简单使用，可以简单的透传 W3C 标准的 TraceContext，详细可以参考 [W3C TraceContext](https://www.w3.org/TR/trace-context/)。
即：将 request 请求中的 `traceparent` 和 `tracestate` 请求头发送到需要被请求的消息头中即可。

### 推荐配置

为了您的应用能够更好的运行在服务网格环境下，我们推荐您在应用中做如下配置：

#### 健康检查

建议给您的应用配置健康检查，以便服务网格能够更好的监控您的应用的健康状态。具体可以参考
[Kubernetes 官方文档](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)。

#### 重试机制

由于 Sidecar 机制的存在，可能存在 Sidecar 在服务之后启动的情况。在此期间，
您的服务可能无法访问外部请求，所以我们建议您在应用中配置重试机制，以便在 Sidecar 启动后，能够自动重试请求。

#### 使用 HTTP 或者 GRPC 作为应用通讯协议

目前我们的服务网格能够非常好的支持 HTTP 和 GRPC 协议，建议您的应用也使用这两种协议，
如果您的应用是其他协议，如 BRPC，则会被降级成 TCP 协议，可能会导致一些功能无法使用。

默认情况下，服务网格会对服务间访问进行 TLS 加密（mTLS），所以我们也不建议在服务间的访问使用 HTTPS 协议（会被当做 TCP 协议处理）。

所以我们建议您的应用使用 HTTP 或者 GRPC 协议。

#### 避免使用外置注册中心机制

虽然我们的产品支持使用注册中心的服务接入服务网格，但这并不是我们推荐的方式，
我们建议您的应用直接使用服务网格（Kubernetes 的 Service）的服务发现机制，以便能够更好的使用服务网格的功能。

也就是说，我们不推荐使用 Spring Cloud 这样的框架开发新的应用，但是这类存量的应用可以被我们产品支持。
如果是 Java 应用，我们推荐使用 Spring Boot 的方式开发应用。

### 服务对外暴露

默认情况下，接入服务网格的应用我们不推荐使用 NodePort 或者 LoadBalancer 等方式直接对外提供服务，这将导致部分服务网格的功能无法使用。

在目前的版本中，我们有两种方式可以对外提供服务：

1. 使用`网格网关`进行对外暴露。
1. 使用`微服务引擎`的`云原生网关`进行对外暴露。

对于`微服务引擎`的`云原生网关`使用方法，请参阅相关文档。

对于`网格网关`的使用方法，也请参阅相关文档。值得注意的是，我们建议将需要对外服务的 VirtualService 独立出来，
不要和其他（如 Service 同名的 VirtualService）的 VirtualService 混合在一起，以便能够更好的管理。
