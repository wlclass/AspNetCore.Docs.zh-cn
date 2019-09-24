---
title: 安全 ASP.NET Core Blazor 服务器应用
author: guardrex
description: 了解如何缓解 Blazor 服务器应用的安全威胁。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/23/2019
uid: security/blazor/server
ms.openlocfilehash: 706f504738d9c6e5af3c368c382424f2e206bcbf
ms.sourcegitcommit: 79eeb17604b536e8f34641d1e6b697fb9a2ee21f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2019
ms.locfileid: "71211708"
---
# <a name="secure-aspnet-core-blazor-server-apps"></a><span data-ttu-id="cb97d-103">安全 ASP.NET Core Blazor 服务器应用</span><span class="sxs-lookup"><span data-stu-id="cb97d-103">Secure ASP.NET Core Blazor Server apps</span></span>

<span data-ttu-id="cb97d-104">作者： [Javier Calvarro 使用](https://github.com/javiercn)</span><span class="sxs-lookup"><span data-stu-id="cb97d-104">By [Javier Calvarro Nelson](https://github.com/javiercn)</span></span>

<span data-ttu-id="cb97d-105">Blazor Server apps 采用有*状态*数据处理模型，其中服务器和客户端保持长期的关系。</span><span class="sxs-lookup"><span data-stu-id="cb97d-105">Blazor Server apps adopt a *stateful* data processing model, where the server and client maintain a long-lived relationship.</span></span> <span data-ttu-id="cb97d-106">持久状态由线路维护，[线路](xref:blazor/state-management)可以跨也可能长期生存期的连接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-106">The persistent state is maintained by a [circuit](xref:blazor/state-management), which can span connections that are also potentially long-lived.</span></span>

<span data-ttu-id="cb97d-107">当用户访问 Blazor 服务器站点时，服务器将在服务器的内存中创建线路。</span><span class="sxs-lookup"><span data-stu-id="cb97d-107">When a user visits a Blazor Server site, the server creates a circuit in the server's memory.</span></span> <span data-ttu-id="cb97d-108">线路向浏览器指示要呈现的内容并响应事件，例如当用户在 UI 中选择一个按钮时。</span><span class="sxs-lookup"><span data-stu-id="cb97d-108">The circuit indicates to the browser what content to render and responds to events, such as when the user selects a button in the UI.</span></span> <span data-ttu-id="cb97d-109">若要执行这些操作，线路会在用户的浏览器和服务器上的 .NET 方法中调用 JavaScript 函数。</span><span class="sxs-lookup"><span data-stu-id="cb97d-109">To perform these actions, a circuit invokes JavaScript functions in the user's browser and .NET methods on the server.</span></span> <span data-ttu-id="cb97d-110">这种基于 JavaScript 的双向交互称为[javascript 互操作（JS 互操作）](xref:blazor/javascript-interop)。</span><span class="sxs-lookup"><span data-stu-id="cb97d-110">This two-way JavaScript-based interaction is referred to as [JavaScript interop (JS interop)](xref:blazor/javascript-interop).</span></span>

<span data-ttu-id="cb97d-111">由于在 Internet 上进行 JS 互操作，并且客户端使用远程浏览器，Blazor Server apps 共享大多数 web 应用安全问题。</span><span class="sxs-lookup"><span data-stu-id="cb97d-111">Because JS interop occurs over the Internet and the client uses a remote browser, Blazor Server apps share most web app security concerns.</span></span> <span data-ttu-id="cb97d-112">本主题介绍对 Blazor 服务器应用的常见威胁，并提供侧重于面向 Internet 的应用的威胁缓解指导。</span><span class="sxs-lookup"><span data-stu-id="cb97d-112">This topic describes common threats to Blazor Server apps and provides threat mitigation guidance focused on Internet-facing apps.</span></span>

<span data-ttu-id="cb97d-113">在受约束的环境（如公司网络或 intranet 内部）中，一些缓解指导原则如下：</span><span class="sxs-lookup"><span data-stu-id="cb97d-113">In constrained environments, such as inside corporate networks or intranets, some of the mitigation guidance either:</span></span>

* <span data-ttu-id="cb97d-114">不适用于受约束的环境中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-114">Doesn't apply in the constrained environment.</span></span>
* <span data-ttu-id="cb97d-115">由于受约束的环境中的安全风险不足，因此不值得实施。</span><span class="sxs-lookup"><span data-stu-id="cb97d-115">Isn't worth the cost to implement because the security risk is low in a constrained environment.</span></span>

## <a name="resource-exhaustion"></a><span data-ttu-id="cb97d-116">资源耗尽</span><span class="sxs-lookup"><span data-stu-id="cb97d-116">Resource exhaustion</span></span>

<span data-ttu-id="cb97d-117">当客户端与服务器交互并导致服务器消耗过多资源时，可能会发生资源耗尽。</span><span class="sxs-lookup"><span data-stu-id="cb97d-117">Resource exhaustion can occur when a client interacts with the server and causes the server to consume excessive resources.</span></span> <span data-ttu-id="cb97d-118">过度消耗资源主要影响：</span><span class="sxs-lookup"><span data-stu-id="cb97d-118">Excessive resource consumption primarily affects:</span></span>

* [<span data-ttu-id="cb97d-119">CPU</span><span class="sxs-lookup"><span data-stu-id="cb97d-119">CPU</span></span>](#cpu)
* [<span data-ttu-id="cb97d-120">内存</span><span class="sxs-lookup"><span data-stu-id="cb97d-120">Memory</span></span>](#memory)
* [<span data-ttu-id="cb97d-121">客户端连接</span><span class="sxs-lookup"><span data-stu-id="cb97d-121">Client connections</span></span>](#client-connections)

<span data-ttu-id="cb97d-122">拒绝服务（DoS）攻击通常会设法排出应用或服务器的资源。</span><span class="sxs-lookup"><span data-stu-id="cb97d-122">Denial of service (DoS) attacks usually seek to exhaust an app or server's resources.</span></span> <span data-ttu-id="cb97d-123">但是，资源耗尽并不一定是对系统的攻击。</span><span class="sxs-lookup"><span data-stu-id="cb97d-123">However, resource exhaustion isn't necessarily the result of an attack on the system.</span></span> <span data-ttu-id="cb97d-124">例如，由于用户需求较高，因此有限资源可能会耗尽。</span><span class="sxs-lookup"><span data-stu-id="cb97d-124">For example, finite resources can be exhausted due to high user demand.</span></span> <span data-ttu-id="cb97d-125">[拒绝服务（DoS）攻击](#denial-of-service-dos-attacks)部分进一步介绍了 DoS。</span><span class="sxs-lookup"><span data-stu-id="cb97d-125">DoS is covered further in the [Denial of service (DoS) attacks](#denial-of-service-dos-attacks) section.</span></span>

<span data-ttu-id="cb97d-126">Blazor 框架之外的资源（例如数据库和文件句柄（用于读取和写入文件））可能还会遇到资源耗尽。</span><span class="sxs-lookup"><span data-stu-id="cb97d-126">Resources external to the Blazor framework, such as databases and file handles (used to read and write files), may also experience resource exhaustion.</span></span> <span data-ttu-id="cb97d-127">有关详细信息，请参阅 <xref:performance/performance-best-practices> 。</span><span class="sxs-lookup"><span data-stu-id="cb97d-127">For more information, see <xref:performance/performance-best-practices>.</span></span>

### <a name="cpu"></a><span data-ttu-id="cb97d-128">CPU</span><span class="sxs-lookup"><span data-stu-id="cb97d-128">CPU</span></span>

<span data-ttu-id="cb97d-129">当一个或多个客户端强制服务器执行密集型 CPU 工作时，就会出现 CPU 消耗。</span><span class="sxs-lookup"><span data-stu-id="cb97d-129">CPU exhaustion can occur when one or more clients force the server to perform intensive CPU work.</span></span>

<span data-ttu-id="cb97d-130">例如，请考虑一个计算*Fibonnacci 号*的 Blazor 服务器应用程序。</span><span class="sxs-lookup"><span data-stu-id="cb97d-130">For example, consider a Blazor Server app that calculates a *Fibonnacci number*.</span></span> <span data-ttu-id="cb97d-131">Fibonnacci 数由 Fibonnacci 序列生成，其中序列中的每个数字都是上述两个数字之和。</span><span class="sxs-lookup"><span data-stu-id="cb97d-131">A Fibonnacci number is produced from a Fibonnacci sequence, where each number in the sequence is the sum of the two preceding numbers.</span></span> <span data-ttu-id="cb97d-132">达到该应答值所需的工作量取决于序列的长度和初始值的大小。</span><span class="sxs-lookup"><span data-stu-id="cb97d-132">The amount of work required to reach the answer depends on the length of the sequence and the size of the initial value.</span></span> <span data-ttu-id="cb97d-133">如果应用不会对客户端的请求施加限制，CPU 密集型计算可能会占用 CPU 时间，并降低其他任务的性能。</span><span class="sxs-lookup"><span data-stu-id="cb97d-133">If the app doesn't place limits on a client's request, the CPU-intensive calculations may dominate the CPU's time and diminish the performance of other tasks.</span></span> <span data-ttu-id="cb97d-134">过度消耗资源是影响可用性的安全问题。</span><span class="sxs-lookup"><span data-stu-id="cb97d-134">Excessive resource consumption is a security concern impacting availability.</span></span>

<span data-ttu-id="cb97d-135">对于所有面向公众的应用，都需要考虑 CPU 消耗。</span><span class="sxs-lookup"><span data-stu-id="cb97d-135">CPU exhaustion is a concern for all public-facing apps.</span></span> <span data-ttu-id="cb97d-136">在常规 web 应用中，请求和连接将作为一项安全措施超时，但 Blazor 服务器应用不提供相同的安全措施。</span><span class="sxs-lookup"><span data-stu-id="cb97d-136">In regular web apps, requests and connections time out as a safeguard, but Blazor Server apps don't provide the same safeguards.</span></span> <span data-ttu-id="cb97d-137">在执行可能占用 CPU 的工作之前，Blazor 服务器应用必须包含适当的检查和限制。</span><span class="sxs-lookup"><span data-stu-id="cb97d-137">Blazor Server apps must include appropriate checks and limits before performing potentially CPU-intensive work.</span></span>

### <a name="memory"></a><span data-ttu-id="cb97d-138">内存</span><span class="sxs-lookup"><span data-stu-id="cb97d-138">Memory</span></span>

<span data-ttu-id="cb97d-139">当一个或多个客户端强制服务器消耗大量内存时，可能会出现内存耗尽。</span><span class="sxs-lookup"><span data-stu-id="cb97d-139">Memory exhaustion can occur when one or more clients force the server to consume a large amount of memory.</span></span>

<span data-ttu-id="cb97d-140">例如，假设有一个 Blazor 的服务器端应用程序，该应用程序具有接受并显示项列表的组件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-140">For example, consider a Blazor-server side app with a component that accepts and displays a list of items.</span></span> <span data-ttu-id="cb97d-141">如果 Blazor 应用不会对允许的项数或向客户端呈现的项数施加限制，则内存密集型处理和呈现可能会将服务器的内存占用到服务器性能的点。</span><span class="sxs-lookup"><span data-stu-id="cb97d-141">If the Blazor app doesn't place limits on the number of items allowed or the number of items rendered back to the client, the memory-intensive processing and rendering may dominate the server's memory to the point where performance of the server suffers.</span></span> <span data-ttu-id="cb97d-142">服务器可能会崩溃或速度太慢，因为似乎已崩溃。</span><span class="sxs-lookup"><span data-stu-id="cb97d-142">The server may crash or slow to the point that it appears to have crashed.</span></span>

<span data-ttu-id="cb97d-143">请考虑以下方案来维护和显示与服务器上可能的内存消耗方案相关的项列表：</span><span class="sxs-lookup"><span data-stu-id="cb97d-143">Consider the following scenario for maintaining and displaying a list of items that pertain to a potential memory exhaustion scenario on the server:</span></span>

* <span data-ttu-id="cb97d-144">`List<MyItem>`属性或字段中的项使用服务器的内存。</span><span class="sxs-lookup"><span data-stu-id="cb97d-144">The items in a `List<MyItem>` property or field use the server's memory.</span></span> <span data-ttu-id="cb97d-145">如果应用允许项列表无限制地增长，则服务器的内存不足。</span><span class="sxs-lookup"><span data-stu-id="cb97d-145">If the app allows the list of items to grow unbounded, there's a risk of the server running out of memory.</span></span> <span data-ttu-id="cb97d-146">内存不足会导致当前会话结束（崩溃），并且该服务器实例中的所有并发会话都将收到内存不足异常。</span><span class="sxs-lookup"><span data-stu-id="cb97d-146">Running out of memory causes the current session to end (crash) and all of the concurrent sessions in that server instance receive an out-of-memory exception.</span></span> <span data-ttu-id="cb97d-147">若要防止这种情况发生，应用程序必须使用在并发用户上施加项限制的数据结构。</span><span class="sxs-lookup"><span data-stu-id="cb97d-147">To prevent this scenario from occurring, the app must use a data structure that imposes an item limit on concurrent users.</span></span>
* <span data-ttu-id="cb97d-148">如果分页方案不用于呈现，则服务器将为用户界面中不可见的对象使用额外内存。</span><span class="sxs-lookup"><span data-stu-id="cb97d-148">If a paging scheme isn't used for rendering, the server uses additional memory for objects that aren't visible in the UI.</span></span> <span data-ttu-id="cb97d-149">如果项目数量没有限制，内存需求可能会耗尽可用的服务器内存。</span><span class="sxs-lookup"><span data-stu-id="cb97d-149">Without a limit on the number of items, memory demands may exhaust the available server memory.</span></span> <span data-ttu-id="cb97d-150">若要防止这种情况，请使用以下方法之一：</span><span class="sxs-lookup"><span data-stu-id="cb97d-150">To prevent this scenario, use one of the following approaches:</span></span>
  * <span data-ttu-id="cb97d-151">在呈现时使用分页列表。</span><span class="sxs-lookup"><span data-stu-id="cb97d-151">Use paginated lists when rendering.</span></span>
  * <span data-ttu-id="cb97d-152">仅显示前100到1000项，并要求用户输入搜索条件以查找超出所显示项的项。</span><span class="sxs-lookup"><span data-stu-id="cb97d-152">Only display the first 100 to 1,000 items and require the user to enter search criteria to find items beyond the items displayed.</span></span>
  * <span data-ttu-id="cb97d-153">对于更高级的渲染方案，实现支持*虚拟化*的列表或网格。</span><span class="sxs-lookup"><span data-stu-id="cb97d-153">For a more advanced rendering scenario, implement lists or grids that support *virtualization*.</span></span> <span data-ttu-id="cb97d-154">使用虚拟化，列表只呈现用户当前可见的项的子集。</span><span class="sxs-lookup"><span data-stu-id="cb97d-154">Using virtualization, lists only render a subset of items currently visible to the user.</span></span> <span data-ttu-id="cb97d-155">当用户与 UI 中的滚动条交互时，组件仅呈现显示所需的项。</span><span class="sxs-lookup"><span data-stu-id="cb97d-155">When the user interacts with the scrollbar in the UI, the component renders only those items required for display.</span></span> <span data-ttu-id="cb97d-156">当前不需要的显示内容可保存在辅助存储中，这是理想的方法。</span><span class="sxs-lookup"><span data-stu-id="cb97d-156">The items that aren't currently required for display can be held in secondary storage, which is the ideal approach.</span></span> <span data-ttu-id="cb97d-157">副标题项还可以保存在内存中，这种情况不太理想。</span><span class="sxs-lookup"><span data-stu-id="cb97d-157">Undisplayed items can also be held in memory, which is less ideal.</span></span>

<span data-ttu-id="cb97d-158">Blazor Server apps 为有状态应用程序（如 WPF、Windows 窗体或 Blazor WebAssembly）的其他 UI 框架提供了类似的编程模型。</span><span class="sxs-lookup"><span data-stu-id="cb97d-158">Blazor Server apps offer a similar programming model to other UI frameworks for stateful apps, such as WPF, Windows Forms, or Blazor WebAssembly.</span></span> <span data-ttu-id="cb97d-159">主要区别在于，在多个 UI 框架中，应用使用的内存属于客户端，仅影响单个客户端。</span><span class="sxs-lookup"><span data-stu-id="cb97d-159">The main difference is that in several of the UI frameworks the memory consumed by the app belongs to the client and only affects that individual client.</span></span> <span data-ttu-id="cb97d-160">例如，Blazor WebAssembly 应用完全在客户端上运行，并且只使用客户端内存资源。</span><span class="sxs-lookup"><span data-stu-id="cb97d-160">For example, a Blazor WebAssembly app runs entirely on the client and only uses client memory resources.</span></span> <span data-ttu-id="cb97d-161">在 Blazor 服务器方案中，应用使用的内存属于服务器，并在服务器实例上的客户端之间共享。</span><span class="sxs-lookup"><span data-stu-id="cb97d-161">In the Blazor Server scenario, the memory consumed by the app belongs to the server and is shared among clients on the server instance.</span></span>

<span data-ttu-id="cb97d-162">服务器端内存需求是所有 Blazor 服务器应用的注意事项。</span><span class="sxs-lookup"><span data-stu-id="cb97d-162">Server-side memory demands are a consideration for all Blazor Server apps.</span></span> <span data-ttu-id="cb97d-163">但是，大多数 web 应用都是无状态的，并且在返回响应时，将释放在处理请求时使用的内存。</span><span class="sxs-lookup"><span data-stu-id="cb97d-163">However, most web apps are stateless, and the memory used while processing a request is released when the response is returned.</span></span> <span data-ttu-id="cb97d-164">作为一般建议，不允许客户端分配未绑定的内存量，就像在任何其他服务器端应用程序中保留客户端连接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-164">As a general recommendation, don't permit clients to allocate an unbound amount of memory as in any other server-side app that persists client connections.</span></span> <span data-ttu-id="cb97d-165">Blazor 服务器应用使用的内存比单个请求持续更长时间。</span><span class="sxs-lookup"><span data-stu-id="cb97d-165">The memory consumed by a Blazor Server app persists for a longer time than a single request.</span></span>

> [!NOTE]
> <span data-ttu-id="cb97d-166">在开发过程中，可以使用探查器或捕获捕获的跟踪来评估客户端的内存需求。</span><span class="sxs-lookup"><span data-stu-id="cb97d-166">During development, a profiler can be used or a trace captured to assess memory demands of clients.</span></span> <span data-ttu-id="cb97d-167">探查器或跟踪不会捕获分配给特定客户端的内存。</span><span class="sxs-lookup"><span data-stu-id="cb97d-167">A profiler or trace won't capture the memory allocated to a specific client.</span></span> <span data-ttu-id="cb97d-168">若要捕获开发期间特定客户端的内存使用情况，请捕获转储，并检查以用户线路为根的所有对象的内存需求。</span><span class="sxs-lookup"><span data-stu-id="cb97d-168">To capture the memory use of a specific client during development, capture a dump and examine the memory demand of all the objects rooted at a user's circuit.</span></span>

### <a name="client-connections"></a><span data-ttu-id="cb97d-169">客户端连接</span><span class="sxs-lookup"><span data-stu-id="cb97d-169">Client connections</span></span>

<span data-ttu-id="cb97d-170">当一个或多个客户端与服务器建立的并发连接太多时，可能会发生连接耗尽，这会阻止其他客户端建立新连接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-170">Connection exhaustion can occur when one or more clients open too many concurrent connections to the server, preventing other clients from establishing new connections.</span></span>

<span data-ttu-id="cb97d-171">Blazor 客户端建立每个会话的单个连接，只要浏览器窗口处于打开状态，就会保持连接打开。</span><span class="sxs-lookup"><span data-stu-id="cb97d-171">Blazor clients establish a single connection per session and keep the connection open for as long as the browser window is open.</span></span> <span data-ttu-id="cb97d-172">维护所有连接的服务器上的要求并不特定于 Blazor 应用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-172">The demands on the server of maintaining all of the connections isn't specific to Blazor apps.</span></span> <span data-ttu-id="cb97d-173">由于连接的持久特性和 Blazor 服务器应用程序的有状态特性，连接用尽对于应用程序的可用性具有更大的风险。</span><span class="sxs-lookup"><span data-stu-id="cb97d-173">Given the persistent nature of the connections and the stateful nature of Blazor Server apps, connection exhaustion is a greater risk to availability of the app.</span></span>

<span data-ttu-id="cb97d-174">默认情况下，Blazor 服务器应用的每个用户的连接数没有限制。</span><span class="sxs-lookup"><span data-stu-id="cb97d-174">By default, there's no limit on the number of connections per user for a Blazor Server app.</span></span> <span data-ttu-id="cb97d-175">如果应用需要连接限制，请采用以下一种或多种方法：</span><span class="sxs-lookup"><span data-stu-id="cb97d-175">If the app requires a connection limit, take one or more of the following approaches:</span></span>

* <span data-ttu-id="cb97d-176">要求身份验证，这自然会限制未经授权的用户连接到应用的能力。</span><span class="sxs-lookup"><span data-stu-id="cb97d-176">Require authentication, which naturally limits the ability of unauthorized users to connect to the app.</span></span> <span data-ttu-id="cb97d-177">要使这种情况生效，用户必须能够在不设置新用户的情况下进行设置。</span><span class="sxs-lookup"><span data-stu-id="cb97d-177">For this scenario to be effective, users must be prevented from provisioning new users at will.</span></span>
* <span data-ttu-id="cb97d-178">限制每个用户的连接数。</span><span class="sxs-lookup"><span data-stu-id="cb97d-178">Limit the number of connections per user.</span></span> <span data-ttu-id="cb97d-179">可以通过以下方法来限制连接的限制。</span><span class="sxs-lookup"><span data-stu-id="cb97d-179">Limiting connections can be accomplished via the following approaches.</span></span> <span data-ttu-id="cb97d-180">请注意允许合法用户访问应用（例如，在基于客户端的 IP 地址建立连接限制时）。</span><span class="sxs-lookup"><span data-stu-id="cb97d-180">Exercise care to allow legitimate users to access the app (for example, when a connection limit is established based on the client's IP address).</span></span>
  * <span data-ttu-id="cb97d-181">在应用级别：</span><span class="sxs-lookup"><span data-stu-id="cb97d-181">At the app level:</span></span>
    * <span data-ttu-id="cb97d-182">终结点路由扩展性。</span><span class="sxs-lookup"><span data-stu-id="cb97d-182">Endpoint routing extensibility.</span></span>
    * <span data-ttu-id="cb97d-183">要求进行身份验证以连接到应用并跟踪每个用户的活动会话。</span><span class="sxs-lookup"><span data-stu-id="cb97d-183">Require authentication to connect to the app and keep track of the active sessions per user.</span></span>
    * <span data-ttu-id="cb97d-184">达到限制时拒绝新会话。</span><span class="sxs-lookup"><span data-stu-id="cb97d-184">Reject new sessions upon reaching a limit.</span></span>
    * <span data-ttu-id="cb97d-185">代理 WebSocket 通过使用代理连接到应用，例如[Azure SignalR 服务](/azure/azure-signalr/signalr-overview)，多路复用从客户端到应用的连接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-185">Proxy WebSocket connections to an app through the use of a proxy, such as the [Azure SignalR Service](/azure/azure-signalr/signalr-overview) that multiplexes connections from clients to an app.</span></span> <span data-ttu-id="cb97d-186">这为应用提供的连接容量比单个客户端可以建立的连接容量大，从而防止客户端耗尽连接到服务器。</span><span class="sxs-lookup"><span data-stu-id="cb97d-186">This provides an app with greater connection capacity than a single client can establish, preventing a client from exhausting the connections to the server.</span></span>
  * <span data-ttu-id="cb97d-187">在服务器级别：在应用前使用代理/网关。</span><span class="sxs-lookup"><span data-stu-id="cb97d-187">At the server level: Use a proxy/gateway in front of the app.</span></span> <span data-ttu-id="cb97d-188">例如， [Azure 前门](/azure/frontdoor/front-door-overview)使你可以定义、管理和监视 web 流量到应用的全局路由。</span><span class="sxs-lookup"><span data-stu-id="cb97d-188">For example, [Azure Front Door](/azure/frontdoor/front-door-overview) enables you to define, manage, and monitor the global routing of web traffic to an app.</span></span>

## <a name="denial-of-service-dos-attacks"></a><span data-ttu-id="cb97d-189">拒绝服务（DoS）攻击</span><span class="sxs-lookup"><span data-stu-id="cb97d-189">Denial of service (DoS) attacks</span></span>

<span data-ttu-id="cb97d-190">拒绝服务（DoS）攻击涉及客户端导致服务器耗尽其一个或多个资源，使应用不可用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-190">Denial of service (DoS) attacks involve a client causing the server to exhaust one or more of its resources making the app unavailable.</span></span> <span data-ttu-id="cb97d-191">Blazor 服务器应用程序包含一些默认限制，并依赖于其他 ASP.NET Core 和 SignalR 限制来防范 DoS 攻击：</span><span class="sxs-lookup"><span data-stu-id="cb97d-191">Blazor Server apps include some default limits and rely on other ASP.NET Core and SignalR limits to protect against DoS attacks:</span></span>

| <span data-ttu-id="cb97d-192">Blazor 服务器应用程序限制</span><span class="sxs-lookup"><span data-stu-id="cb97d-192">Blazor Server app limit</span></span>                            | <span data-ttu-id="cb97d-193">描述</span><span class="sxs-lookup"><span data-stu-id="cb97d-193">Description</span></span> | <span data-ttu-id="cb97d-194">默认</span><span class="sxs-lookup"><span data-stu-id="cb97d-194">Default</span></span> |
| ------------------------------------------------------- | ----------- | ------- |
| `CircuitOptions.DisconnectedCircuitMaxRetained`         | <span data-ttu-id="cb97d-195">给定服务器一次在内存中保留的最大断开连接电路数。</span><span class="sxs-lookup"><span data-stu-id="cb97d-195">Maximum number of disconnected circuits that a given server holds in memory at a time.</span></span> | <span data-ttu-id="cb97d-196">100</span><span class="sxs-lookup"><span data-stu-id="cb97d-196">100</span></span> |
| `CircuitOptions.DisconnectedCircuitRetentionPeriod`     | <span data-ttu-id="cb97d-197">断开连接线路之前在内存中保留的最大时间。</span><span class="sxs-lookup"><span data-stu-id="cb97d-197">Maximum amount of time a disconnected circuit is held in memory before being torn down.</span></span> | <span data-ttu-id="cb97d-198">3分钟</span><span class="sxs-lookup"><span data-stu-id="cb97d-198">3 minutes</span></span> |
| `CircuitOptions.JSInteropDefaultCallTimeout`            | <span data-ttu-id="cb97d-199">服务器在超时异步 JavaScript 函数调用之前等待的最长时间。</span><span class="sxs-lookup"><span data-stu-id="cb97d-199">Maximum amount of time the server waits before timing out an asynchronous JavaScript function invocation.</span></span> | <span data-ttu-id="cb97d-200">1 分钟</span><span class="sxs-lookup"><span data-stu-id="cb97d-200">1 minute</span></span> |
| `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches` | <span data-ttu-id="cb97d-201">服务器在给定时间为每个线路保存在内存中的未确认呈现批处理的最大数目，以支持可靠的重新连接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-201">Maximum number of unacknowledged render batches the server keeps in memory per circuit at a given time to support robust reconnection.</span></span> <span data-ttu-id="cb97d-202">达到该限制后，服务器将停止生成新的呈现批，直到客户端确认了一个或多个批处理。</span><span class="sxs-lookup"><span data-stu-id="cb97d-202">After reaching the limit, the server stops producing new render batches until one or more batches have been acknowledged by the client.</span></span> | <span data-ttu-id="cb97d-203">10</span><span class="sxs-lookup"><span data-stu-id="cb97d-203">10</span></span> |


| <span data-ttu-id="cb97d-204">SignalR 和 ASP.NET Core 限制</span><span class="sxs-lookup"><span data-stu-id="cb97d-204">SignalR and ASP.NET Core limit</span></span>             | <span data-ttu-id="cb97d-205">描述</span><span class="sxs-lookup"><span data-stu-id="cb97d-205">Description</span></span> | <span data-ttu-id="cb97d-206">默认</span><span class="sxs-lookup"><span data-stu-id="cb97d-206">Default</span></span> |
| ------------------------------------------ | ----------- | ------- |
| `CircuitOptions.MaximumReceiveMessageSize` | <span data-ttu-id="cb97d-207">单个消息的消息大小。</span><span class="sxs-lookup"><span data-stu-id="cb97d-207">Message size for an individual message.</span></span> | <span data-ttu-id="cb97d-208">32 KB</span><span class="sxs-lookup"><span data-stu-id="cb97d-208">32 KB</span></span> |

## <a name="interactions-with-the-browser-client"></a><span data-ttu-id="cb97d-209">与浏览器（客户端）的交互</span><span class="sxs-lookup"><span data-stu-id="cb97d-209">Interactions with the browser (client)</span></span>

<span data-ttu-id="cb97d-210">客户端通过 JS 互操作事件调度与服务器交互，并呈现完成。</span><span class="sxs-lookup"><span data-stu-id="cb97d-210">A client interacts with the server through JS interop event dispatching and render completion.</span></span> <span data-ttu-id="cb97d-211">在 JavaScript 和 .NET 之间，JS 互操作通信是双向的：</span><span class="sxs-lookup"><span data-stu-id="cb97d-211">JS interop communication goes both ways between JavaScript and .NET:</span></span>

* <span data-ttu-id="cb97d-212">浏览器事件以异步方式从客户端调度到服务器。</span><span class="sxs-lookup"><span data-stu-id="cb97d-212">Browser events are dispatched from the client to the server in an asynchronous fashion.</span></span>
* <span data-ttu-id="cb97d-213">服务器根据需要以异步方式 rerendering UI。</span><span class="sxs-lookup"><span data-stu-id="cb97d-213">The server responds asynchronously rerendering the UI as necessary.</span></span>

### <a name="javascript-functions-invoked-from-net"></a><span data-ttu-id="cb97d-214">从 .NET 调用的 JavaScript 函数</span><span class="sxs-lookup"><span data-stu-id="cb97d-214">JavaScript functions invoked from .NET</span></span>

<span data-ttu-id="cb97d-215">对于从 .NET 方法到 JavaScript 的调用：</span><span class="sxs-lookup"><span data-stu-id="cb97d-215">For calls from .NET methods to JavaScript:</span></span>

* <span data-ttu-id="cb97d-216">所有调用都具有可配置的超时时间，在此之后<xref:System.OperationCanceledException> ，将返回到调用方。</span><span class="sxs-lookup"><span data-stu-id="cb97d-216">All invocations have a configurable timeout after which they fail, returning a <xref:System.OperationCanceledException> to the caller.</span></span>
  * <span data-ttu-id="cb97d-217">调用的默认超时值（`CircuitOptions.JSInteropDefaultCallTimeout`）为一分钟。</span><span class="sxs-lookup"><span data-stu-id="cb97d-217">There's a default timeout for the calls (`CircuitOptions.JSInteropDefaultCallTimeout`) of one minute.</span></span> <span data-ttu-id="cb97d-218">若要配置此限制， <xref:blazor/javascript-interop#harden-js-interop-calls>请参阅。</span><span class="sxs-lookup"><span data-stu-id="cb97d-218">To configure this limit, see <xref:blazor/javascript-interop#harden-js-interop-calls>.</span></span>
  * <span data-ttu-id="cb97d-219">可以提供取消标记以按调用控制取消。</span><span class="sxs-lookup"><span data-stu-id="cb97d-219">A cancellation token can be provided to control the cancellation on a per-call basis.</span></span> <span data-ttu-id="cb97d-220">如果提供了取消标记，则使用默认调用超时，如果有可能，则依赖于对客户端的任何调用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-220">Rely on the default call timeout where possible and time-bound any call to the client if a cancellation token is provided.</span></span>
* <span data-ttu-id="cb97d-221">JavaScript 调用的结果不能受信任。</span><span class="sxs-lookup"><span data-stu-id="cb97d-221">The result of a JavaScript call can't be trusted.</span></span> <span data-ttu-id="cb97d-222">在浏览器中运行的 Blazor 应用客户端将搜索要调用的 JavaScript 函数。</span><span class="sxs-lookup"><span data-stu-id="cb97d-222">The Blazor app client running in the browser searches for the JavaScript function to invoke.</span></span> <span data-ttu-id="cb97d-223">调用函数，并生成结果或错误。</span><span class="sxs-lookup"><span data-stu-id="cb97d-223">The function is invoked, and either the result or an error is produced.</span></span> <span data-ttu-id="cb97d-224">恶意客户端可以尝试：</span><span class="sxs-lookup"><span data-stu-id="cb97d-224">A malicious client can attempt to:</span></span>
  * <span data-ttu-id="cb97d-225">通过从 JavaScript 函数返回错误，导致应用中出现问题。</span><span class="sxs-lookup"><span data-stu-id="cb97d-225">Cause an issue in the app by returning an error from the JavaScript function.</span></span>
  * <span data-ttu-id="cb97d-226">通过从 JavaScript 函数返回意外的结果，在服务器上引发意外的行为。</span><span class="sxs-lookup"><span data-stu-id="cb97d-226">Induce an unintended behavior on the server by returning an unexpected result from the JavaScript function.</span></span>

<span data-ttu-id="cb97d-227">请采取以下预防措施来防范前述方案：</span><span class="sxs-lookup"><span data-stu-id="cb97d-227">Take the following precautions to guard against the preceding scenarios:</span></span>

* <span data-ttu-id="cb97d-228">将 JS 互操作调用包装在[try-catch](/dotnet/csharp/language-reference/keywords/try-catch)语句中，以考虑在调用期间可能发生的错误。</span><span class="sxs-lookup"><span data-stu-id="cb97d-228">Wrap JS interop calls within [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) statements to account for errors that might occur during the invocations.</span></span> <span data-ttu-id="cb97d-229">有关详细信息，请参阅 <xref:blazor/handle-errors#javascript-interop> 。</span><span class="sxs-lookup"><span data-stu-id="cb97d-229">For more information, see <xref:blazor/handle-errors#javascript-interop>.</span></span>
* <span data-ttu-id="cb97d-230">在执行任何操作之前，验证从 JS 互操作调用返回的数据，包括错误消息。</span><span class="sxs-lookup"><span data-stu-id="cb97d-230">Validate data returned from JS interop invocations, including error messages, before taking any action.</span></span>

### <a name="net-methods-invoked-from-the-browser"></a><span data-ttu-id="cb97d-231">从浏览器调用的 .NET 方法</span><span class="sxs-lookup"><span data-stu-id="cb97d-231">.NET methods invoked from the browser</span></span>

<span data-ttu-id="cb97d-232">不要信任从 JavaScript 到 .NET 方法的调用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-232">Don't trust calls from JavaScript to .NET methods.</span></span> <span data-ttu-id="cb97d-233">向 JavaScript 公开 .NET 方法时，请考虑 .NET 方法的调用方式：</span><span class="sxs-lookup"><span data-stu-id="cb97d-233">When a .NET method is exposed to JavaScript, consider how the .NET method is invoked:</span></span>

* <span data-ttu-id="cb97d-234">像对应用程序的公共终结点一样对待公开给 JavaScript 的任何 .NET 方法。</span><span class="sxs-lookup"><span data-stu-id="cb97d-234">Treat any .NET method exposed to JavaScript as you would a public endpoint to the app.</span></span>
  * <span data-ttu-id="cb97d-235">验证输入。</span><span class="sxs-lookup"><span data-stu-id="cb97d-235">Validate input.</span></span>
    * <span data-ttu-id="cb97d-236">确保值在预期范围内。</span><span class="sxs-lookup"><span data-stu-id="cb97d-236">Ensure that values are within expected ranges.</span></span>
    * <span data-ttu-id="cb97d-237">确保用户有权执行所请求的操作。</span><span class="sxs-lookup"><span data-stu-id="cb97d-237">Ensure that the user has permission to perform the action requested.</span></span>
  * <span data-ttu-id="cb97d-238">在 .NET 方法调用中，不要分配过多的资源。</span><span class="sxs-lookup"><span data-stu-id="cb97d-238">Don't allocate an excessive quantity of resources as part of the .NET method invocation.</span></span> <span data-ttu-id="cb97d-239">例如，对 CPU 和内存使用执行检查并施加限制。</span><span class="sxs-lookup"><span data-stu-id="cb97d-239">For example, perform checks and place limits on CPU and memory use.</span></span>
  * <span data-ttu-id="cb97d-240">请考虑可向 JavaScript 客户端公开静态和实例方法。</span><span class="sxs-lookup"><span data-stu-id="cb97d-240">Take into account that static and instance methods can be exposed to JavaScript clients.</span></span> <span data-ttu-id="cb97d-241">避免在不同的会话中共享状态，除非设计调用具有相应约束的共享状态。</span><span class="sxs-lookup"><span data-stu-id="cb97d-241">Avoid sharing state across sessions unless the design calls for sharing state with appropriate constraints.</span></span>
    * <span data-ttu-id="cb97d-242">对于通过`DotNetReference`依赖关系注入（DI）最初创建的对象公开的实例方法，应将这些对象注册为范围对象。</span><span class="sxs-lookup"><span data-stu-id="cb97d-242">For instance methods exposed through `DotNetReference` objects that are originally created through dependency injection (DI), the objects should be registered as scoped objects.</span></span> <span data-ttu-id="cb97d-243">这适用于 Blazor 服务器应用使用的任何 DI 服务。</span><span class="sxs-lookup"><span data-stu-id="cb97d-243">This applies to any DI service that the Blazor Server app uses.</span></span>
    * <span data-ttu-id="cb97d-244">对于静态方法，请避免建立不能作用于客户端的状态，除非应用在服务器实例上的所有用户之间通过设计显式共享状态。</span><span class="sxs-lookup"><span data-stu-id="cb97d-244">For static methods, avoid establishing state that can't be scoped to the client unless the app is explicitly sharing state by-design across all users on a server instance.</span></span>
  * <span data-ttu-id="cb97d-245">避免将用户提供的数据传入参数到 JavaScript 调用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-245">Avoid passing user-supplied data in parameters to JavaScript calls.</span></span> <span data-ttu-id="cb97d-246">如果绝对需要在参数中传递数据，请确保 JavaScript 代码处理数据，而不引入[跨站点脚本（XSS）](#cross-site-scripting-xss)漏洞。</span><span class="sxs-lookup"><span data-stu-id="cb97d-246">If passing data in parameters is absolutely required, ensure that the JavaScript code handles passing the data without introducing [Cross-site scripting (XSS)](#cross-site-scripting-xss) vulnerabilities.</span></span> <span data-ttu-id="cb97d-247">例如，不要通过设置`innerHTML`元素的属性将用户提供的数据写入文档对象模型（DOM）。</span><span class="sxs-lookup"><span data-stu-id="cb97d-247">For example, don't write user-supplied data to the Document Object Model (DOM) by setting the `innerHTML` property of an element.</span></span> <span data-ttu-id="cb97d-248">请考虑使用[内容安全策略（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)来`eval`禁用和其他不安全的 JavaScript 基元。</span><span class="sxs-lookup"><span data-stu-id="cb97d-248">Consider using [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) to disable `eval` and other unsafe JavaScript primitives.</span></span>
* <span data-ttu-id="cb97d-249">避免在框架的调度实现之上实现 .NET 调用的自定义调度。</span><span class="sxs-lookup"><span data-stu-id="cb97d-249">Avoid implementing custom dispatching of .NET invocations on top of the framework's dispatching implementation.</span></span> <span data-ttu-id="cb97d-250">将 .NET 方法公开给浏览器是一种高级方案，不建议用于常规 Blazor 开发。</span><span class="sxs-lookup"><span data-stu-id="cb97d-250">Exposing .NET methods to the browser is an advanced scenario, not recommended for general Blazor development.</span></span>

### <a name="events"></a><span data-ttu-id="cb97d-251">事件</span><span class="sxs-lookup"><span data-stu-id="cb97d-251">Events</span></span>

<span data-ttu-id="cb97d-252">事件提供 Blazor 服务器应用程序的入口点。</span><span class="sxs-lookup"><span data-stu-id="cb97d-252">Events provide an entry point to a Blazor Server app.</span></span> <span data-ttu-id="cb97d-253">用于保护 web 应用中的终结点的相同规则适用于 Blazor 服务器应用中的事件处理。</span><span class="sxs-lookup"><span data-stu-id="cb97d-253">The same rules for safeguarding endpoints in web apps apply to event handling in Blazor Server apps.</span></span> <span data-ttu-id="cb97d-254">恶意客户端可以将其希望发送的任何数据发送为事件的负载。</span><span class="sxs-lookup"><span data-stu-id="cb97d-254">A malicious client can send any data it wishes to send as the payload for an event.</span></span>

<span data-ttu-id="cb97d-255">例如:</span><span class="sxs-lookup"><span data-stu-id="cb97d-255">For example:</span></span>

* <span data-ttu-id="cb97d-256">的更改事件`<select>`可能会发送一个值，该值不在应用提供给客户端的选项中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-256">A change event for a `<select>` could send a value that isn't within the options that the app presented to the client.</span></span>
* <span data-ttu-id="cb97d-257">`<input>`可以跳过客户端验证，将任何文本数据发送到服务器。</span><span class="sxs-lookup"><span data-stu-id="cb97d-257">An `<input>` could send any text data to the server, bypassing client-side validation.</span></span>

<span data-ttu-id="cb97d-258">应用必须验证应用处理的任何事件的数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-258">The app must validate the data for any event that the app handles.</span></span> <span data-ttu-id="cb97d-259">Blazor 框架[窗体组件](xref:blazor/forms-validation)执行基本验证。</span><span class="sxs-lookup"><span data-stu-id="cb97d-259">The Blazor framework [forms components](xref:blazor/forms-validation) perform basic validations.</span></span> <span data-ttu-id="cb97d-260">如果应用使用自定义窗体组件，则必须编写自定义代码以根据需要验证事件数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-260">If the app uses custom forms components, custom code must be written to validate event data as appropriate.</span></span>

<span data-ttu-id="cb97d-261">Blazor 服务器事件是异步的，因此在应用程序有时间通过生成新的呈现方式之前，可以将多个事件分派给服务器。</span><span class="sxs-lookup"><span data-stu-id="cb97d-261">Blazor Server events are asynchronous, so multiple events can be dispatched to the server before the app has time to react by producing a new render.</span></span> <span data-ttu-id="cb97d-262">这有一些需要注意的安全问题。</span><span class="sxs-lookup"><span data-stu-id="cb97d-262">This has some security implications to consider.</span></span> <span data-ttu-id="cb97d-263">必须在事件处理程序中执行限制应用程序中的客户端操作，而不是依赖于当前呈现的视图状态。</span><span class="sxs-lookup"><span data-stu-id="cb97d-263">Limiting client actions in the app must be performed inside event handlers and not depend on the current rendered view state.</span></span>

<span data-ttu-id="cb97d-264">假设有一个计数器组件，该组件应该允许用户最多递增三次计数器。</span><span class="sxs-lookup"><span data-stu-id="cb97d-264">Consider a counter component that should allow a user to increment a counter a maximum of three times.</span></span> <span data-ttu-id="cb97d-265">根据的值`count`，有条件地递增计数器的按钮：</span><span class="sxs-lookup"><span data-stu-id="cb97d-265">The button to increment the counter is conditionally based on the value of `count`:</span></span>

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

<span data-ttu-id="cb97d-266">在框架生成此组件的新呈现之前，客户端可以调度一个或多个增量事件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-266">A client can dispatch one or more increment events before the framework produces a new render of this component.</span></span> <span data-ttu-id="cb97d-267">因此，用户`count`可以在*三次*后递增，因为用户界面不会快速删除该按钮。</span><span class="sxs-lookup"><span data-stu-id="cb97d-267">The result is that the `count` can be incremented *over three times* by the user because the button isn't removed by the UI quickly enough.</span></span> <span data-ttu-id="cb97d-268">下面的示例显示了实现三个`count`增量限制的正确方法：</span><span class="sxs-lookup"><span data-stu-id="cb97d-268">The correct way to achieve the limit of three `count` increments is shown in the following example:</span></span>

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        if (count < 3)
        {
            count++;
        }
    }
}
```

<span data-ttu-id="cb97d-269">通过在处理`if (count < 3) { ... }`程序内添加检查，可以根据当前应用`count`程序状态进行增量决定。</span><span class="sxs-lookup"><span data-stu-id="cb97d-269">By adding the `if (count < 3) { ... }` check inside the handler, the decision to increment `count` is based on the current app state.</span></span> <span data-ttu-id="cb97d-270">该决策不基于 UI 的状态，这可能是暂时过时的。</span><span class="sxs-lookup"><span data-stu-id="cb97d-270">The decision isn't based on the state of the UI as it was in the previous example, which might be temporarily stale.</span></span>

### <a name="guard-against-multiple-dispatches"></a><span data-ttu-id="cb97d-271">防范多个派单</span><span class="sxs-lookup"><span data-stu-id="cb97d-271">Guard against multiple dispatches</span></span>

<span data-ttu-id="cb97d-272">如果事件回调调用长时间运行的操作（如从外部服务或数据库提取数据），请考虑使用临界。</span><span class="sxs-lookup"><span data-stu-id="cb97d-272">If an event callback invokes a long running operation, such as fetching data from an external service or database, consider using a guard.</span></span> <span data-ttu-id="cb97d-273">当操作正在进行时，该保护可阻止用户在执行多个操作的同时进行可视反馈。</span><span class="sxs-lookup"><span data-stu-id="cb97d-273">The guard can prevent the user from queueing up multiple operations while the operation is in progress with visual feedback.</span></span> <span data-ttu-id="cb97d-274">以下组件代码将设置`isLoading`为`true` ， `GetForecastAsync`同时从服务器获取数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-274">The following component code sets `isLoading` to `true` while `GetForecastAsync` obtains data from the server.</span></span> <span data-ttu-id="cb97d-275">当`isLoading` 为`true`时，该按钮在 UI 中处于禁用状态：</span><span class="sxs-lookup"><span data-stu-id="cb97d-275">While `isLoading` is `true`, the button is disabled in the UI:</span></span>

```cshtml
@page "/fetchdata"
@using BlazorServerSample.Data
@inject WeatherForecastService ForecastService

<button disabled="@isLoading" @onclick="UpdateForecasts">Update</button>

@code {
    private bool isLoading;
    private WeatherForecast[] forecasts;

    private async Task UpdateForecasts()
    {
        if (!isLoading)
        {
            isLoading = true;
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
            isLoading = false;
        }
    }
}
```

### <a name="cancel-early-and-avoid-use-after-dispose"></a><span data-ttu-id="cb97d-276">尽早取消并避免使用-dispose</span><span class="sxs-lookup"><span data-stu-id="cb97d-276">Cancel early and avoid use-after-dispose</span></span>

<span data-ttu-id="cb97d-277">除了使用防范[多个派单](#guard-against-multiple-dispatches)部分中所述的临界外，还应考虑使用<xref:System.Threading.CancellationToken>在释放组件时取消长时间运行的操作。</span><span class="sxs-lookup"><span data-stu-id="cb97d-277">In addition to using a guard as described in the [Guard against multiple dispatches](#guard-against-multiple-dispatches) section, consider using a <xref:System.Threading.CancellationToken> to cancel long-running operations when the component is disposed.</span></span> <span data-ttu-id="cb97d-278">此方法的优点在于避免在组件中*使用后期*处理：</span><span class="sxs-lookup"><span data-stu-id="cb97d-278">This approach has the added benefit of avoiding *use-after-dispose* in components:</span></span>

```cshtml
@implements IDisposable

...

@code {
    private readonly CancellationTokenSource TokenSource = 
        new CancellationTokenSource();

    private async Task UpdateForecasts()
    {
        ...

        forecasts = await ForecastService.GetForecastAsync(DateTime.Now, 
            TokenSource.Token);

        if (TokenSource.Token.IsCancellationRequested)
        {
           return;
        }

        ...
    }

    public void Dispose()
    {
        CancellationTokenSource.Cancel();
    }
}
```

### <a name="avoid-events-that-produce-large-amounts-of-data"></a><span data-ttu-id="cb97d-279">避免产生大量数据的事件</span><span class="sxs-lookup"><span data-stu-id="cb97d-279">Avoid events that produce large amounts of data</span></span>

<span data-ttu-id="cb97d-280">某些 DOM 事件（如`oninput`或`onscroll`）可能会生成大量的数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-280">Some DOM events, such as `oninput` or `onscroll`, can produce a large amount of data.</span></span> <span data-ttu-id="cb97d-281">避免在 Blazor 服务器应用中使用这些事件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-281">Avoid using these events in Blazor server apps.</span></span>

## <a name="additional-security-guidance"></a><span data-ttu-id="cb97d-282">其他安全指南</span><span class="sxs-lookup"><span data-stu-id="cb97d-282">Additional security guidance</span></span>

<span data-ttu-id="cb97d-283">ASP.NET Core 应用的安全指南适用于 Blazor 服务器应用，包括以下各节：</span><span class="sxs-lookup"><span data-stu-id="cb97d-283">The guidance for securing ASP.NET Core apps apply to Blazor Server apps and are covered in the following sections:</span></span>

* [<span data-ttu-id="cb97d-284">日志记录和敏感数据</span><span class="sxs-lookup"><span data-stu-id="cb97d-284">Logging and sensitive data</span></span>](#logging-and-sensitive-data)
* [<span data-ttu-id="cb97d-285">通过 HTTPS 保护传输中的信息</span><span class="sxs-lookup"><span data-stu-id="cb97d-285">Protect information in transit with HTTPS</span></span>](#protect-information-in-transit-with-https)
* <span data-ttu-id="cb97d-286">[跨站点脚本（XSS）](#cross-site-scripting-xss)）</span><span class="sxs-lookup"><span data-stu-id="cb97d-286">[Cross-site scripting (XSS)](#cross-site-scripting-xss))</span></span>
* [<span data-ttu-id="cb97d-287">跨源保护</span><span class="sxs-lookup"><span data-stu-id="cb97d-287">Cross-origin protection</span></span>](#cross-origin-protection)
* [<span data-ttu-id="cb97d-288">单击-点击劫持</span><span class="sxs-lookup"><span data-stu-id="cb97d-288">Click-jacking</span></span>](#click-jacking)
* [<span data-ttu-id="cb97d-289">打开重定向</span><span class="sxs-lookup"><span data-stu-id="cb97d-289">Open redirects</span></span>](#open-redirects)

### <a name="logging-and-sensitive-data"></a><span data-ttu-id="cb97d-290">日志记录和敏感数据</span><span class="sxs-lookup"><span data-stu-id="cb97d-290">Logging and sensitive data</span></span>

<span data-ttu-id="cb97d-291">客户端和服务器之间的 JS 互操作交互记录在具有<xref:Microsoft.Extensions.Logging.ILogger>实例的服务器日志中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-291">JS interop interactions between the client and server are recorded in the server's logs with <xref:Microsoft.Extensions.Logging.ILogger> instances.</span></span> <span data-ttu-id="cb97d-292">Blazor 避免记录敏感信息，如实际事件或 JS 互操作输入和输出。</span><span class="sxs-lookup"><span data-stu-id="cb97d-292">Blazor avoids logging sensitive information, such as actual events or JS interop inputs and outputs.</span></span>

<span data-ttu-id="cb97d-293">服务器上发生错误时，框架会通知客户端，并泪水会话。</span><span class="sxs-lookup"><span data-stu-id="cb97d-293">When an error occurs on the server, the framework notifies the client and tears down the session.</span></span> <span data-ttu-id="cb97d-294">默认情况下，客户端会收到一般错误消息，可以在浏览器的开发人员工具中查看。</span><span class="sxs-lookup"><span data-stu-id="cb97d-294">By default, the client receives a generic error message that can be seen in the browser's developer tools.</span></span>

<span data-ttu-id="cb97d-295">客户端错误不包括调用堆栈，并且不提供有关错误原因的详细信息，但服务器日志包含这样的信息。</span><span class="sxs-lookup"><span data-stu-id="cb97d-295">The client-side error doesn't include the callstack and doesn't provide detail on the cause of the error, but server logs do contain such information.</span></span> <span data-ttu-id="cb97d-296">出于开发目的，可以通过启用详细错误，使敏感错误信息可供客户端使用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-296">For development purposes, sensitive error information can be made available to the client by enabling detailed errors.</span></span>

<span data-ttu-id="cb97d-297">使用以下内容启用详细错误：</span><span class="sxs-lookup"><span data-stu-id="cb97d-297">Enable detailed errors with:</span></span>

* <span data-ttu-id="cb97d-298">`CircuitOptions.DetailedErrors`。</span><span class="sxs-lookup"><span data-stu-id="cb97d-298">`CircuitOptions.DetailedErrors`.</span></span>
* <span data-ttu-id="cb97d-299">`DetailedErrors`配置键。</span><span class="sxs-lookup"><span data-stu-id="cb97d-299">`DetailedErrors` configuration key.</span></span> <span data-ttu-id="cb97d-300">例如，将`ASPNETCORE_DETAILEDERRORS`环境变量设置为的`true`值。</span><span class="sxs-lookup"><span data-stu-id="cb97d-300">For example, set the `ASPNETCORE_DETAILEDERRORS` environment variable to a value of `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="cb97d-301">向 Internet 上的客户端公开错误信息是应始终避免的安全风险。</span><span class="sxs-lookup"><span data-stu-id="cb97d-301">Exposing error information to clients on the Internet is a security risk that should always be avoided.</span></span>

### <a name="protect-information-in-transit-with-https"></a><span data-ttu-id="cb97d-302">通过 HTTPS 保护传输中的信息</span><span class="sxs-lookup"><span data-stu-id="cb97d-302">Protect information in transit with HTTPS</span></span>

<span data-ttu-id="cb97d-303">Blazor 服务器使用 SignalR 在客户端和服务器之间进行通信。</span><span class="sxs-lookup"><span data-stu-id="cb97d-303">Blazor Server uses SignalR for communication between the client and the server.</span></span> <span data-ttu-id="cb97d-304">Blazor 服务器通常使用 SignalR 协商的传输，这通常是 Websocket。</span><span class="sxs-lookup"><span data-stu-id="cb97d-304">Blazor Server normally uses the transport that SignalR negotiates, which is typically WebSockets.</span></span>

<span data-ttu-id="cb97d-305">Blazor 服务器不确保在服务器和客户端之间发送的数据的完整性和保密性。</span><span class="sxs-lookup"><span data-stu-id="cb97d-305">Blazor Server doesn't ensure the integrity and confidentiality of the data sent between the server and the client.</span></span> <span data-ttu-id="cb97d-306">始终使用 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="cb97d-306">Always use HTTPS.</span></span>

### <a name="cross-site-scripting-xss"></a><span data-ttu-id="cb97d-307">跨站点脚本（XSS）</span><span class="sxs-lookup"><span data-stu-id="cb97d-307">Cross-site scripting (XSS)</span></span>

<span data-ttu-id="cb97d-308">跨站点脚本（XSS）允许未经授权的参与方在浏览器的上下文中执行任意逻辑。</span><span class="sxs-lookup"><span data-stu-id="cb97d-308">Cross-site scripting (XSS) allows an unauthorized party to execute arbitrary logic in the context of the browser.</span></span> <span data-ttu-id="cb97d-309">受损的应用可能会在客户端上运行任意代码。</span><span class="sxs-lookup"><span data-stu-id="cb97d-309">A compromised app could potentially run arbitrary code on the client.</span></span> <span data-ttu-id="cb97d-310">此漏洞可用于对服务器执行大量恶意操作：</span><span class="sxs-lookup"><span data-stu-id="cb97d-310">The vulnerability could be used to potentially perform a number of malicious actions against the server:</span></span>

* <span data-ttu-id="cb97d-311">向服务器发送虚假/无效事件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-311">Dispatch fake/invalid events to the server.</span></span>
* <span data-ttu-id="cb97d-312">调度失败/呈现完成无效。</span><span class="sxs-lookup"><span data-stu-id="cb97d-312">Dispatch fail/invalid render completions.</span></span>
* <span data-ttu-id="cb97d-313">避免调度呈现完成。</span><span class="sxs-lookup"><span data-stu-id="cb97d-313">Avoid dispatching render completions.</span></span>
* <span data-ttu-id="cb97d-314">从 JavaScript 向 .NET 调度互操作调用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-314">Dispatch interop calls from JavaScript to .NET.</span></span>
* <span data-ttu-id="cb97d-315">修改从 .NET 到 JavaScript 的互操作调用的响应。</span><span class="sxs-lookup"><span data-stu-id="cb97d-315">Modify the response of interop calls from .NET to JavaScript.</span></span>
* <span data-ttu-id="cb97d-316">避免将 .NET 调度到 JS 互操作结果。</span><span class="sxs-lookup"><span data-stu-id="cb97d-316">Avoid dispatching .NET to JS interop results.</span></span>

<span data-ttu-id="cb97d-317">Blazor 服务器框架采用一些步骤来防范前面的威胁：</span><span class="sxs-lookup"><span data-stu-id="cb97d-317">The Blazor Server framework takes steps to protect against some of the preceding threats:</span></span>

* <span data-ttu-id="cb97d-318">如果客户端未确认呈现批次，则停止生成新的 UI 更新。</span><span class="sxs-lookup"><span data-stu-id="cb97d-318">Stops producing new UI updates if the client isn't acknowledging render batches.</span></span> <span data-ttu-id="cb97d-319">配置了`CircuitOptions.MaxBufferedUnacknowledgedRenderBatches`。</span><span class="sxs-lookup"><span data-stu-id="cb97d-319">Configured with `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches`.</span></span>
* <span data-ttu-id="cb97d-320">在一分钟后无需收到客户端响应的情况下，任何 .NET 到 JavaScript 调用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-320">Times out any .NET to JavaScript call after one minute without receiving a response from the client.</span></span> <span data-ttu-id="cb97d-321">配置了`CircuitOptions.JSInteropDefaultCallTimeout`。</span><span class="sxs-lookup"><span data-stu-id="cb97d-321">Configured with `CircuitOptions.JSInteropDefaultCallTimeout`.</span></span>
* <span data-ttu-id="cb97d-322">对在 JS 互操作期间来自浏览器的所有输入执行基本验证：</span><span class="sxs-lookup"><span data-stu-id="cb97d-322">Performs basic validation on all input coming from the browser during JS interop:</span></span>
  * <span data-ttu-id="cb97d-323">.NET 引用是有效的，并且是 .NET 方法所需的类型。</span><span class="sxs-lookup"><span data-stu-id="cb97d-323">.NET references are valid and of the type expected by the .NET method.</span></span>
  * <span data-ttu-id="cb97d-324">数据的格式不正确。</span><span class="sxs-lookup"><span data-stu-id="cb97d-324">The data isn't malformed.</span></span>
  * <span data-ttu-id="cb97d-325">有效负载中存在方法的参数的正确数目。</span><span class="sxs-lookup"><span data-stu-id="cb97d-325">The correct number of arguments for the method are present in the payload.</span></span>
  * <span data-ttu-id="cb97d-326">参数或结果可以在调用方法之前正确地进行反序列化。</span><span class="sxs-lookup"><span data-stu-id="cb97d-326">The arguments or result can be deserialized correctly before invoking the method.</span></span>
* <span data-ttu-id="cb97d-327">在来自浏览器的事件的所有输入中执行基本验证：</span><span class="sxs-lookup"><span data-stu-id="cb97d-327">Performs basic validation in all input coming from the browser from dispatched events:</span></span>
  * <span data-ttu-id="cb97d-328">事件的类型有效。</span><span class="sxs-lookup"><span data-stu-id="cb97d-328">The event has a valid type.</span></span>
  * <span data-ttu-id="cb97d-329">可对事件的数据进行反序列化。</span><span class="sxs-lookup"><span data-stu-id="cb97d-329">The data for the event can be deserialized.</span></span>
  * <span data-ttu-id="cb97d-330">存在与事件关联的事件处理程序。</span><span class="sxs-lookup"><span data-stu-id="cb97d-330">There's an event handler associated with the event.</span></span>

<span data-ttu-id="cb97d-331">除了框架实现的保护机制，开发人员还必须对应用程序进行编码以防范威胁并采取适当的措施：</span><span class="sxs-lookup"><span data-stu-id="cb97d-331">In addition to the safeguards that the framework implements, the app must be coded by the developer to safeguard against threats and take appropriate actions:</span></span>

* <span data-ttu-id="cb97d-332">处理事件时始终验证数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-332">Always validate data when handling events.</span></span>
* <span data-ttu-id="cb97d-333">接收无效数据时采取适当的措施：</span><span class="sxs-lookup"><span data-stu-id="cb97d-333">Take appropriate action upon receiving invalid data:</span></span>
  * <span data-ttu-id="cb97d-334">忽略数据并返回。</span><span class="sxs-lookup"><span data-stu-id="cb97d-334">Ignore the data and return.</span></span> <span data-ttu-id="cb97d-335">这允许应用继续处理请求。</span><span class="sxs-lookup"><span data-stu-id="cb97d-335">This allows the app to continue processing requests.</span></span>
  * <span data-ttu-id="cb97d-336">如果应用确定输入是非法的且无法由合法的客户端生成，将引发异常。</span><span class="sxs-lookup"><span data-stu-id="cb97d-336">If the app determines that the input is illegitimate and couldn't be produced by legitimate client, throw an exception.</span></span> <span data-ttu-id="cb97d-337">引发异常泪水线路并结束会话。</span><span class="sxs-lookup"><span data-stu-id="cb97d-337">Throwing an exception tears down the circuit and ends the session.</span></span>
* <span data-ttu-id="cb97d-338">不要信任日志中包含的呈现批处理完成所提供的错误消息。</span><span class="sxs-lookup"><span data-stu-id="cb97d-338">Don't trust the error message provided by render batch completions included in the logs.</span></span> <span data-ttu-id="cb97d-339">此错误是由客户端提供的，并且通常无法信任，因为客户端可能会受到威胁。</span><span class="sxs-lookup"><span data-stu-id="cb97d-339">The error is provided by the client and can't generally be trusted, as the client might be compromised.</span></span>
* <span data-ttu-id="cb97d-340">不要在 JavaScript 和 .NET 方法之间的任意方向上信任对 JS 互操作调用的输入。</span><span class="sxs-lookup"><span data-stu-id="cb97d-340">Don't trust the input on JS interop calls in either direction between JavaScript and .NET methods.</span></span>
* <span data-ttu-id="cb97d-341">应用负责验证参数和结果的内容是否有效，即使参数或结果已正确反序列化。</span><span class="sxs-lookup"><span data-stu-id="cb97d-341">The app is responsible for validating that the content of arguments and results are valid, even if the arguments or results are correctly deserialized.</span></span>

<span data-ttu-id="cb97d-342">要使 XSS 漏洞存在，应用必须将用户输入合并到呈现的页面中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-342">For a XSS vulnerability to exist, the app must incorporate user input in the rendered page.</span></span> <span data-ttu-id="cb97d-343">Blazor 服务器组件执行编译时步骤，其中， *razor*文件中的标记将转换为过程C#逻辑。</span><span class="sxs-lookup"><span data-stu-id="cb97d-343">Blazor Server components execute a compile-time step where the markup in a *.razor* file is transformed into procedural C# logic.</span></span> <span data-ttu-id="cb97d-344">在运行时， C#逻辑将生成描述元素、文本和子组件的*呈现树*。</span><span class="sxs-lookup"><span data-stu-id="cb97d-344">At runtime, the C# logic builds a *render tree* describing the elements, text, and child components.</span></span> <span data-ttu-id="cb97d-345">此操作通过一系列 JavaScript 指令应用于浏览器的 DOM （或者在预呈现时序列化为 HTML）：</span><span class="sxs-lookup"><span data-stu-id="cb97d-345">This is applied to the browser's DOM via a sequence of JavaScript instructions (or is serialized to HTML in the case of prerendering):</span></span>

* <span data-ttu-id="cb97d-346">通过普通 Razor 语法（例如`@someStringValue`）呈现的用户输入不会公开 XSS 漏洞，因为 Razor 语法通过只能写入文本的命令添加到 DOM。</span><span class="sxs-lookup"><span data-stu-id="cb97d-346">User input rendered via normal Razor syntax (for example, `@someStringValue`) doesn't expose a XSS vulnerability because the Razor syntax is added to the DOM via commands that can only write text.</span></span> <span data-ttu-id="cb97d-347">即使该值包含 HTML 标记，该值也会显示为静态文本。</span><span class="sxs-lookup"><span data-stu-id="cb97d-347">Even if the value includes HTML markup, the value is displayed as static text.</span></span> <span data-ttu-id="cb97d-348">预呈现时，输出是 HTML 编码的，它还将内容显示为静态文本。</span><span class="sxs-lookup"><span data-stu-id="cb97d-348">When prerendering, the output is HTML-encoded, which also displays the content as static text.</span></span>
* <span data-ttu-id="cb97d-349">不允许使用脚本标记，也不应将其包含在应用的组件呈现树中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-349">Script tags aren't allowed and shouldn't be included in the app's component render tree.</span></span> <span data-ttu-id="cb97d-350">如果某个脚本标记包含在组件的标记中，则会生成编译时错误。</span><span class="sxs-lookup"><span data-stu-id="cb97d-350">If a script tag is included in a component's markup, a compile-time error is generated.</span></span>
* <span data-ttu-id="cb97d-351">组件作者无需使用 Razor C#即可在中创作组件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-351">Component authors can author components in C# without using Razor.</span></span> <span data-ttu-id="cb97d-352">组件作者负责发出输出时使用正确的 Api。</span><span class="sxs-lookup"><span data-stu-id="cb97d-352">The component author is responsible for using the correct APIs when emitting output.</span></span> <span data-ttu-id="cb97d-353">例如，使用`builder.AddContent(0, someUserSuppliedString)`而*不* `builder.AddMarkupContent(0, someUserSuppliedString)`是，因为后者可能会创建 XSS 漏洞。</span><span class="sxs-lookup"><span data-stu-id="cb97d-353">For example, use `builder.AddContent(0, someUserSuppliedString)` and *not* `builder.AddMarkupContent(0, someUserSuppliedString)`, as the latter could create a XSS vulnerability.</span></span>

<span data-ttu-id="cb97d-354">作为防范 XSS 攻击的一部分，请考虑实施 XSS 缓解，如[内容安全策略（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)。</span><span class="sxs-lookup"><span data-stu-id="cb97d-354">As part of protecting against XSS attacks, consider implementing XSS mitigations, such as [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP).</span></span>

<span data-ttu-id="cb97d-355">有关详细信息，请参阅 <xref:security/cross-site-scripting> 。</span><span class="sxs-lookup"><span data-stu-id="cb97d-355">For more information, see <xref:security/cross-site-scripting>.</span></span>

### <a name="cross-origin-protection"></a><span data-ttu-id="cb97d-356">跨源保护</span><span class="sxs-lookup"><span data-stu-id="cb97d-356">Cross-origin protection</span></span>

<span data-ttu-id="cb97d-357">跨域攻击涉及到针对服务器执行操作的不同源中的客户端。</span><span class="sxs-lookup"><span data-stu-id="cb97d-357">Cross-origin attacks involve a client from a different origin performing an action against the server.</span></span> <span data-ttu-id="cb97d-358">恶意操作通常是 GET 请求或窗体 POST （跨站点请求伪造，CSRF），但也可以打开恶意 WebSocket。</span><span class="sxs-lookup"><span data-stu-id="cb97d-358">The malicious action is typically a GET request or a form POST (Cross-Site Request Forgery, CSRF), but opening a malicious WebSocket is also possible.</span></span> <span data-ttu-id="cb97d-359">Blazor 服务器应用[通过使用集线器协议提供的任何其他 SignalR 应用提供相同的保证](xref:signalr/security)：</span><span class="sxs-lookup"><span data-stu-id="cb97d-359">Blazor Server apps offer [the same guarantees that any other SignalR app using the hub protocol offer](xref:signalr/security):</span></span>

* <span data-ttu-id="cb97d-360">可以跨源访问 Blazor 服务器应用，除非采取其他措施来阻止它。</span><span class="sxs-lookup"><span data-stu-id="cb97d-360">Blazor Server apps can be accessed cross-origin unless additional measures are taken to prevent it.</span></span> <span data-ttu-id="cb97d-361">若要禁用跨域访问，请在终结点中禁用 cors，方法是将 cors 中间件添加到管道`DisableCorsAttribute` ，将添加到 Blazor 终结点元数据，或通过[为跨域资源配置 SignalR 来限制允许来源集共享](xref:signalr/security#cross-origin-resource-sharing)。</span><span class="sxs-lookup"><span data-stu-id="cb97d-361">To disable cross-origin access, either disable CORS in the endpoint by adding the CORS middleware to the pipeline and adding the `DisableCorsAttribute` to the Blazor endpoint metadata or limit the set of allowed origins by [configuring SignalR for cross-origin resource sharing](xref:signalr/security#cross-origin-resource-sharing).</span></span>
* <span data-ttu-id="cb97d-362">如果启用了 CORS，则可能需要执行额外的步骤来保护应用，具体情况视 CORS 配置而定。</span><span class="sxs-lookup"><span data-stu-id="cb97d-362">If CORS is enabled, extra steps might be required to protect the app depending on the CORS configuration.</span></span> <span data-ttu-id="cb97d-363">如果已全局启用 cors，则可以在调用`DisableCorsAttribute` `hub.MapBlazorHub()`后将元数据添加到终结点元数据，从而为 Blazor 服务器中心禁用 cors。</span><span class="sxs-lookup"><span data-stu-id="cb97d-363">If CORS is globally enabled, CORS can be disabled for the Blazor Server hub by adding the `DisableCorsAttribute` metadata to the endpoint metadata after calling `hub.MapBlazorHub()`.</span></span>

<span data-ttu-id="cb97d-364">有关详细信息，请参阅 <xref:security/anti-request-forgery> 。</span><span class="sxs-lookup"><span data-stu-id="cb97d-364">For more information, see <xref:security/anti-request-forgery>.</span></span>

### <a name="click-jacking"></a><span data-ttu-id="cb97d-365">单击-点击劫持</span><span class="sxs-lookup"><span data-stu-id="cb97d-365">Click-jacking</span></span>

<span data-ttu-id="cb97d-366">单击 "-点击劫持" 会将站点作为`<iframe>`站点中的站点从不同的来源呈现，以便诱使用户在受攻击的站点上执行操作。</span><span class="sxs-lookup"><span data-stu-id="cb97d-366">Click-jacking involves rendering a site as an `<iframe>` inside a site from a different origin in order to trick the user into performing actions on the site under attack.</span></span>

<span data-ttu-id="cb97d-367">若要保护应用程序不`<iframe>`会呈现在中，请使用[内容安全策略（CSP](https://developer.mozilla.org/docs/Web/HTTP/CSP) `X-Frame-Options` ）和标头。</span><span class="sxs-lookup"><span data-stu-id="cb97d-367">To protect an app from rendering inside of an `<iframe>`, use [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) and the `X-Frame-Options` header.</span></span> <span data-ttu-id="cb97d-368">有关详细信息，请[参阅 MDN web 文档：X 框架-选项](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options)。</span><span class="sxs-lookup"><span data-stu-id="cb97d-368">For more information, see [MDN web docs: X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options).</span></span>

### <a name="open-redirects"></a><span data-ttu-id="cb97d-369">打开重定向</span><span class="sxs-lookup"><span data-stu-id="cb97d-369">Open redirects</span></span>

<span data-ttu-id="cb97d-370">当 Blazor 服务器应用会话启动时，服务器将对作为启动会话的一部分发送的 Url 执行基本验证。</span><span class="sxs-lookup"><span data-stu-id="cb97d-370">When a Blazor Server app session starts, the server performs basic validation of the URLs sent as part of starting the session.</span></span> <span data-ttu-id="cb97d-371">在建立线路之前，框架将检查基 URL 是否为当前 URL 的父级。</span><span class="sxs-lookup"><span data-stu-id="cb97d-371">The framework checks that the base URL is a parent of the current URL before establishing the circuit.</span></span> <span data-ttu-id="cb97d-372">此框架不会执行其他检查。</span><span class="sxs-lookup"><span data-stu-id="cb97d-372">No additional checks are performed by the framework.</span></span>

<span data-ttu-id="cb97d-373">当用户在客户端上选择某个链接时，该链接的 URL 将发送到服务器，这将确定要执行的操作。</span><span class="sxs-lookup"><span data-stu-id="cb97d-373">When a user selects a link on the client, the URL for the link is sent to the server, which determines what action to take.</span></span> <span data-ttu-id="cb97d-374">例如，应用可能会执行客户端导航，或向浏览器指示浏览器将定位到新位置。</span><span class="sxs-lookup"><span data-stu-id="cb97d-374">For example, the app may perform a client-side navigation or indicate to the browser to go to the new location.</span></span>

<span data-ttu-id="cb97d-375">组件还可以通过使用`NavigationManager`以编程方式触发导航请求。</span><span class="sxs-lookup"><span data-stu-id="cb97d-375">Components can also trigger navigation requests programatically through the use of `NavigationManager`.</span></span> <span data-ttu-id="cb97d-376">在这种情况下，应用可能会执行客户端导航，或向浏览器指示浏览器将跳到新位置。</span><span class="sxs-lookup"><span data-stu-id="cb97d-376">In such scenarios, the app might perform a client-side navigation or indicate to the browser to go to the new location.</span></span>

<span data-ttu-id="cb97d-377">组件必须：</span><span class="sxs-lookup"><span data-stu-id="cb97d-377">Components must:</span></span>

* <span data-ttu-id="cb97d-378">避免在导航调用参数中使用用户输入。</span><span class="sxs-lookup"><span data-stu-id="cb97d-378">Avoid using user input as part of the navigation call arguments.</span></span>
* <span data-ttu-id="cb97d-379">验证参数以确保应用允许目标。</span><span class="sxs-lookup"><span data-stu-id="cb97d-379">Validate arguments to ensure that the target is allowed by the app.</span></span>

<span data-ttu-id="cb97d-380">否则，恶意用户可能会强制浏览器转向攻击者控制的站点。</span><span class="sxs-lookup"><span data-stu-id="cb97d-380">Otherwise, a malicious user can force the browser to go to an attacker-controlled site.</span></span> <span data-ttu-id="cb97d-381">在这种情况下，攻击者会在调用`NavigationManager.Navigate`方法的过程中使用某些用户输入来对应用进行技巧。</span><span class="sxs-lookup"><span data-stu-id="cb97d-381">In this scenario, the attacker tricks the app into using some user input as part of the invocation of the `NavigationManager.Navigate` method.</span></span>

<span data-ttu-id="cb97d-382">此建议也适用于在应用程序中呈现链接时的情况：</span><span class="sxs-lookup"><span data-stu-id="cb97d-382">This advice also applies when rendering links as part of the app:</span></span>

* <span data-ttu-id="cb97d-383">如果可能，请使用相对链接。</span><span class="sxs-lookup"><span data-stu-id="cb97d-383">If possible, use relative links.</span></span>
* <span data-ttu-id="cb97d-384">验证绝对链接目标是否有效，然后再将它们包含在页中。</span><span class="sxs-lookup"><span data-stu-id="cb97d-384">Validate that absolute link destinations are valid before including them in a page.</span></span>

<span data-ttu-id="cb97d-385">有关详细信息，请参阅 <xref:security/preventing-open-redirects> 。</span><span class="sxs-lookup"><span data-stu-id="cb97d-385">For more information, see <xref:security/preventing-open-redirects>.</span></span>

## <a name="authentication-and-authorization"></a><span data-ttu-id="cb97d-386">身份验证和授权</span><span class="sxs-lookup"><span data-stu-id="cb97d-386">Authentication and authorization</span></span>

<span data-ttu-id="cb97d-387">有关身份验证和授权的指导， <xref:security/blazor/index>请参阅。</span><span class="sxs-lookup"><span data-stu-id="cb97d-387">For guidance on authentication and authorization, see <xref:security/blazor/index>.</span></span>

## <a name="security-checklist"></a><span data-ttu-id="cb97d-388">安全清单</span><span class="sxs-lookup"><span data-stu-id="cb97d-388">Security checklist</span></span>

<span data-ttu-id="cb97d-389">以下安全注意事项列表并不全面：</span><span class="sxs-lookup"><span data-stu-id="cb97d-389">The following list of security considerations isn't comprehensive:</span></span>

* <span data-ttu-id="cb97d-390">验证来自事件的参数。</span><span class="sxs-lookup"><span data-stu-id="cb97d-390">Validate arguments from events.</span></span>
* <span data-ttu-id="cb97d-391">通过 JS 互操作调用验证输入和结果。</span><span class="sxs-lookup"><span data-stu-id="cb97d-391">Validate inputs and results from JS interop calls.</span></span>
* <span data-ttu-id="cb97d-392">避免对 .NET 到 JS 互操作调用使用（或预先验证）用户输入。</span><span class="sxs-lookup"><span data-stu-id="cb97d-392">Avoid using (or validate beforehand) user input for .NET to JS interop calls.</span></span>
* <span data-ttu-id="cb97d-393">阻止客户端分配未绑定的内存量。</span><span class="sxs-lookup"><span data-stu-id="cb97d-393">Prevent the client from allocating an unbound amount of memory.</span></span>
  * <span data-ttu-id="cb97d-394">组件中的数据。</span><span class="sxs-lookup"><span data-stu-id="cb97d-394">Data within the component.</span></span>
  * <span data-ttu-id="cb97d-395">`DotNetObject`返回到客户端的引用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-395">`DotNetObject` references returned to the client.</span></span>
* <span data-ttu-id="cb97d-396">防范多个派单。</span><span class="sxs-lookup"><span data-stu-id="cb97d-396">Guard against multiple dispatches.</span></span>
* <span data-ttu-id="cb97d-397">释放组件时，取消长时间运行的操作。</span><span class="sxs-lookup"><span data-stu-id="cb97d-397">Cancel long-running operations when the component is disposed.</span></span>
* <span data-ttu-id="cb97d-398">避免产生大量数据的事件。</span><span class="sxs-lookup"><span data-stu-id="cb97d-398">Avoid events that produce large amounts of data.</span></span>
* <span data-ttu-id="cb97d-399">避免使用用户输入作为对的调用`NavigationManager.Navigate`的一部分，并先针对一组允许的来源验证 url 的用户输入。</span><span class="sxs-lookup"><span data-stu-id="cb97d-399">Avoid using user input as part of calls to `NavigationManager.Navigate` and validate user input for URLs against a set of allowed origins first if unavoidable.</span></span>
* <span data-ttu-id="cb97d-400">不要基于 UI 的状态做出授权决策，只需从组件状态进行决策。</span><span class="sxs-lookup"><span data-stu-id="cb97d-400">Don't make authorization decisions based on the state of the UI but only from component state.</span></span>
* <span data-ttu-id="cb97d-401">请考虑使用[内容安全策略（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)来防范 XSS 攻击。</span><span class="sxs-lookup"><span data-stu-id="cb97d-401">Consider using [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) to protect against XSS attacks.</span></span>
* <span data-ttu-id="cb97d-402">请考虑使用 CSP 和[X 帧选项](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options)来防范单击-点击劫持。</span><span class="sxs-lookup"><span data-stu-id="cb97d-402">Consider using CSP and [X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) to protect against click-jacking.</span></span>
* <span data-ttu-id="cb97d-403">当启用 CORS 或显式禁用 Blazor 应用的 CORS 时，请确保 CORS 设置适用。</span><span class="sxs-lookup"><span data-stu-id="cb97d-403">Ensure CORS settings are appropriate when enabling CORS or explicitly disable CORS for Blazor apps.</span></span>
* <span data-ttu-id="cb97d-404">进行测试，以确保 Blazor 应用的服务器端限制提供可接受的用户体验，而无需接受的风险级别。</span><span class="sxs-lookup"><span data-stu-id="cb97d-404">Test to ensure that the server-side limits for the Blazor app provide an acceptable user experience without unacceptable levels of risk.</span></span>