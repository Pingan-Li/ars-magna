# 字体处理流程
##

```C++
#0 0x7f6ddf9e938f base::debug::CollectStackTrace()
#1 0x7f6ddf73782a base::debug::StackTrace::StackTrace()
#2 0x7f6ddf7377e5 base::debug::StackTrace::StackTrace()
#3 0x7f6ddf9e8e5c base::debug::(anonymous namespace)::StackDumpSignalHandler()
#4 0x7f6da0c3ea00 (/usr/lib64/libc.so.6+0x3e9ff)
#5 0x7f6da0c28899 __GI_abort
#6 0x7f6db3352b67 blink::FontBuilder::FontBuilder()
#7 0x7f6db33bd0c8 blink::StyleResolver::InitialStyleForElement()
#8 0x7f6db33bcca8 blink::StyleResolver::StyleForViewport()
#9 0x7f6db3564a64 blink::Document::Initialize()
#10 0x7f6db3b2d8c3 blink::LocalDOMWindow::InstallNewDocument()
#11 0x7f6db4a2e5ea blink::DocumentLoader::CommitNavigation()
#12 0x7f6db4a63d95 blink::FrameLoader::CommitDocumentLoader()
#13 0x7f6db4a639c0 blink::FrameLoader::Init()
#14 0x7f6db3b57039 blink::LocalFrame::Init()
#15 0x7f6db3d0946f blink::WebLocalFrameImpl::InitializeCoreFrameInternal()
#16 0x7f6db3d08d58 blink::WebLocalFrameImpl::InitializeCoreFrame()
#17 0x7f6db3d087ef blink::WebLocalFrameImpl::CreateMainFrame()
#18 0x7f6db3d08603 blink::WebLocalFrame::CreateMainFrame()
#19 0x7f6dd87440cb content::RenderFrameImpl::CreateMainFrame()
#20 0x7f6dd87c3371 content::RenderViewImpl::Initialize()
#21 0x7f6dd87c3e18 content::RenderViewImpl::Create()
#22 0x7f6dd86bca6f content::AgentSchedulingGroup::CreateView()
#23 0x7f6dd5bb7f2c content::mojom::AgentSchedulingGroupStubDispatch::Accept()
#24 0x7f6dd86be2e0 content::mojom::AgentSchedulingGroupStub<>::Accept()
#25 0x7f6ddf39c167 mojo::InterfaceEndpointClient::HandleValidatedMessage()
#26 0x7f6ddf39b999 mojo::InterfaceEndpointClient::HandleIncomingMessageThunk::Accept()
#27 0x7f6ddf3b420b mojo::MessageDispatcher::Accept()
#28 0x7f6ddf39dcaf mojo::InterfaceEndpointClient::HandleIncomingMessage()
#29 0x7f6ddd7d49fa IPC::(anonymous namespace)::ChannelAssociatedGroupController::AcceptOnEndpointThread()
#30 0x7f6ddd7c990e base::internal::FunctorTraits<>::Invoke<>()
#31 0x7f6ddd7c97b6 base::internal::InvokeHelper<>::MakeItSo<>()
#32 0x7f6ddd7c9723 _ZN4base8internal7InvokerINS0_9BindStateIMN3IPC12_GLOBAL__N_132ChannelAssociatedGroupControllerEFvN4mojo7MessageEEJ13scoped_refptrIS5_ES7_EEEFvvEE7RunImplIS9_NSt4__Cr5tupleIJSB_S7_EEEJLm0ELm1EEEEvOT_OT0_NSG_16integer_sequenceImJXspT1_EEEE
#33 0x7f6ddd7c962c base::internal::Invoker<>::RunOnce()
#34 0x7f6ddf6e2d91 _ZNO4base12OnceCallbackIFvvEE3RunEv
#35 0x7f6ddf8d3c76 base::TaskAnnotator::RunTaskImpl()
#36 0x7f6ddf92b730 base::TaskAnnotator::RunTask<>()
#37 0x7f6ddf92b4d4 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWorkImpl()
#38 0x7f6ddf92abce base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#39 0x7f6ddf92b6b0 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#40 0x7f6ddf7b1e8f base::MessagePumpDefault::Run()
#41 0x7f6ddf92bc32 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::Run()
#42 0x7f6ddf863ef9 base::RunLoop::Run()
#43 0x7f6dd87d3f15 content::RendererMain()
#44 0x7f6dd8cfc009 content::RunZygote()
#45 0x7f6dd8cfc8e6 content::RunOtherNamedProcessTypeMain()
#46 0x7f6dd8cfdb17 content::ContentMainRunnerImpl::Run()
#47 0x7f6dd8cfa3ad content::RunContentProcess()
#48 0x7f6dd8cfad2a content::ContentMain()
#49 0x55a37ae6982a ChromeMain
#50 0x55a37ae69652 main
#51 0x7f6da0c29510 __libc_start_call_main
#52 0x7f6da0c295c9 __libc_start_main_alias_2
#53 0x55a37ae6956a _start
```

```C++
#0 0x7f1f4a9e938f base::debug::CollectStackTrace()
#1 0x7f1f4a73782a base::debug::StackTrace::StackTrace()
#2 0x7f1f4a7377e5 base::debug::StackTrace::StackTrace()
#3 0x7f1f4a9e8e5c base::debug::(anonymous namespace)::StackDumpSignalHandler()
#4 0x7f1f0bc3ea00 (/usr/lib64/libc.so.6+0x3e9ff)
#5 0x7f1f0bc28899 __GI_abort
#6 0x7f1f1e354933 blink::FontBuilder::CreateFont()
#7 0x7f1f1e3a8dae blink::StyleCascade::ApplyHighPriority()
#8 0x7f1f1e3a84fd blink::StyleCascade::Apply()
#9 0x7f1f1e3bfb48 blink::StyleResolver::CascadeAndApplyMatchedProperties()
#10 0x7f1f1e3bdfde blink::StyleResolver::ApplyBaseStyle()
#11 0x7f1f1e3bd552 blink::StyleResolver::ResolveStyle()
#12 0x7f1f1e61202f blink::Element::OriginalStyleForLayoutObject()
#13 0x7f1f1e611d8a blink::Element::StyleForLayoutObject()
#14 0x7f1f1e61336c blink::Element::RecalcOwnStyle()
#15 0x7f1f1e612821 blink::Element::RecalcStyle()
#16 0x7f1f1e44d59b blink::StyleEngine::RecalcStyle()
#17 0x7f1f1e44e63a blink::StyleEngine::RecalcStyle()
#18 0x7f1f1e44eb07 blink::StyleEngine::UpdateStyleAndLayoutTree()
#19 0x7f1f1e562747 blink::Document::UpdateStyle()
#20 0x7f1f1e561c54 blink::Document::UpdateStyleAndLayoutTreeForThisDocument()
#21 0x7f1f1e55ea54 blink::Document::UpdateStyleAndLayoutTree()
#22 0x7f1f1e55ea26 blink::Document::UpdateStyleAndLayoutTree()
#23 0x7f1f1e576cc7 blink::Document::FinishedParsing()
#24 0x7f1f20221c70 blink::HTMLConstructionSite::FinishedParsing()
#25 0x7f1f202846f0 blink::HTMLTreeBuilder::Finished()
#26 0x7f1f20230eff blink::HTMLDocumentParser::end()
#27 0x7f1f2022d65e blink::HTMLDocumentParser::AttemptToRunDeferredScriptsAndEnd()
#28 0x7f1f2022d153 blink::HTMLDocumentParser::PrepareToStopParsing()
#29 0x7f1f2022f3a0 blink::HTMLDocumentParser::AttemptToEnd()
#30 0x7f1f20231146 blink::HTMLDocumentParser::Finish()
#31 0x7f1f1fa278c5 blink::DocumentLoader::FinishedLoading()
#32 0x7f1f1fa2aa4b blink::DocumentLoader::StartLoadingResponse()
#33 0x7f1f1fa2efc8 blink::DocumentLoader::CommitNavigation()
#34 0x7f1f1fa63d85 blink::FrameLoader::CommitDocumentLoader()
#35 0x7f1f1fa67815 blink::FrameLoader::CommitNavigation()
#36 0x7f1f1ed0b3c9 blink::WebLocalFrameImpl::CommitNavigation()
#37 0x7f1f43562c6f content::RenderFrameImpl::SynchronouslyCommitAboutBlankForBug778318()
#38 0x7f1f43561dcd content::RenderFrameImpl::BeginNavigation()
#39 0x7f1f1eccac55 blink::LocalFrameClientImpl::BeginNavigation()
#40 0x7f1f1fa66684 blink::FrameLoader::StartNavigation()
#41 0x7f1f1ef21e1c blink::HTMLFrameOwnerElement::LoadOrRedirectSubframe()
#42 0x7f1f1ef1dea4 blink::HTMLFrameElementBase::OpenURL()
#43 0x7f1f1ef1e8ba blink::HTMLFrameElementBase::SetNameAndOpenURL()
#44 0x7f1f1ef1ea32 blink::HTMLFrameElementBase::DidNotifySubtreeInsertionsToDocument()
#45 0x7f1f1e53db3a blink::ContainerNode::NotifyNodeInserted()
#46 0x7f1f1e53b43f blink::ContainerNode::ParserAppendChild()
#47 0x7f1f202244aa blink::Insert()
#48 0x7f1f2021dc45 blink::ExecuteInsertTask()
#49 0x7f1f2021db06 blink::HTMLConstructionSite::ExecuteTask()
#50 0x7f1f2021efbe blink::HTMLConstructionSite::ExecuteQueuedTasks()
#51 0x7f1f20274dde blink::HTMLTreeBuilder::ConstructTree()
#52 0x7f1f2022fbf9 blink::HTMLDocumentParser::ConstructTreeFromHTMLToken()
#53 0x7f1f2022e692 blink::HTMLDocumentParser::PumpTokenizer()
#54 0x7f1f2022d35c blink::HTMLDocumentParser::PumpTokenizerIfPossible()
#55 0x7f1f2022dceb blink::HTMLDocumentParser::DeferredPumpTokenizerIfPossible()
#56 0x7f1f2023950f base::internal::FunctorTraits<>::Invoke<>()
#57 0x7f1f20239421 base::internal::InvokeHelper<>::MakeItSo<>()
#58 0x7f1f202393a2 _ZN4base8internal7InvokerINS0_9BindStateIMN5blink18HTMLDocumentParserEFvvEJN5cppgc8internal15BasicPersistentIS4_NS8_22StrongPersistentPolicyENS8_20IgnoreLocationPolicyENS8_22DisabledCheckingPolicyEEEEEEFvvEE7RunImplIS6_NSt4__Cr5tupleIJSD_EEEJLm0EEEEvOT_OT0_NSI_16integer_sequenceImJXspT1_EEEE
#59 0x7f1f202392dc base::internal::Invoker<>::RunOnce()
#60 0x7f1f1de436a1 _ZNO4base12OnceCallbackIFvvEE3RunEv
#61 0x7f1f1de4362d WTF::ThreadCheckingCallbackWrapper<>::RunInternal()
#62 0x7f1f1de42373 WTF::ThreadCheckingCallbackWrapper<>::Run()
#63 0x7f1f1de42e8f base::internal::FunctorTraits<>::Invoke<>()
#64 0x7f1f1de42da1 base::internal::InvokeHelper<>::MakeItSo<>()
#65 0x7f1f1de42d22 _ZN4base8internal7InvokerINS0_9BindStateIMN3WTF29ThreadCheckingCallbackWrapperINS_12OnceCallbackIFvvEEES6_EEFvvEJNSt4__Cr10unique_ptrIS8_NSB_14default_deleteIS8_EEEEEEES6_E7RunImplISA_NSB_5tupleIJSF_EEEJLm0EEEEvOT_OT0_NSB_16integer_sequenceImJXspT1_EEEE
#66 0x7f1f1de42c5c base::internal::Invoker<>::RunOnce()
#67 0x7f1f4a6e2d91 _ZNO4base12OnceCallbackIFvvEE3RunEv
#68 0x7f1f4a8d3c76 base::TaskAnnotator::RunTaskImpl()
#69 0x7f1f4a92b730 base::TaskAnnotator::RunTask<>()
#70 0x7f1f4a92b4d4 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWorkImpl()
#71 0x7f1f4a92abce base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#72 0x7f1f4a92b6b0 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#73 0x7f1f4a7b1e8f base::MessagePumpDefault::Run()
#74 0x7f1f4a92bc32 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::Run()
#75 0x7f1f4a863ef9 base::RunLoop::Run()
#76 0x7f1f435d3f15 content::RendererMain()
#77 0x7f1f43afc009 content::RunZygote()
#78 0x7f1f43afc8e6 content::RunOtherNamedProcessTypeMain()
#79 0x7f1f43afdb17 content::ContentMainRunnerImpl::Run()
#80 0x7f1f43afa3ad content::RunContentProcess()
#81 0x7f1f43afad2a content::ContentMain()
#82 0x55711e73e82a ChromeMain
#83 0x55711e73e652 main
#84 0x7f1f0bc29510 __libc_start_call_main
#85 0x7f1f0bc295c9 __libc_start_main_alias_2
#86 0x55711e73e56a _start
```


Font::DrawText 255


```C++
#0 0x7ff93d7e938f base::debug::CollectStackTrace()
#1 0x7ff93d53782a base::debug::StackTrace::StackTrace()
#2 0x7ff93d5377e5 base::debug::StackTrace::StackTrace()
#3 0x7ff93d7e8e5c base::debug::(anonymous namespace)::StackDumpSignalHandler()
#4 0x7ff8fea3ea00 (/usr/lib64/libc.so.6+0x3e9ff)
#5 0x7ff8fea28899 __GI_abort
#6 0x7ff90b0245e3 blink::Font::DrawText()
#7 0x7ff90b381412 _ZZN5blink15GraphicsContext16DrawTextInternalINS_23NGTextFragmentPaintInfoEEEvRKNS_4FontERKT_RKN3gfx6PointFEiRKNS_12AutoDarkModeEENKUlRKN2cc10PaintFlagsEE_clESJ_
#8 0x7ff90b38126e _ZN5blink15GraphicsContext14DrawTextPassesIZNS0_16DrawTextInternalINS_23NGTextFragmentPaintInfoEEEvRKNS_4FontERKT_RKN3gfx6PointFEiRKNS_12AutoDarkModeEEUlRKN2cc10PaintFlagsEE_EEvSG_S9_
#9 0x7ff90b37ff15 blink::GraphicsContext::DrawTextInternal<>()
#10 0x7ff90b37c14d blink::GraphicsContext::DrawText()
#11 0x7ff912ab4bef blink::NGTextPainter::PaintInternalFragment<>()
#12 0x7ff912ab3e07 blink::NGTextPainter::PaintInternal<>()
#13 0x7ff912ab18bb blink::NGTextPainter::Paint()
#14 0x7ff912ab04b6 blink::NGTextFragmentPainter::Paint()
#15 0x7ff912a98fd4 blink::NGBoxFragmentPainter::PaintTextItem()
#16 0x7ff912a95933 blink::NGBoxFragmentPainter::PaintInlineItems()
#17 0x7ff912a99b0d blink::NGBoxFragmentPainter::PaintLineBoxChildItems()
#18 0x7ff912a96657 blink::NGBoxFragmentPainter::PaintLineBoxChildren()
#19 0x7ff912a95e0f blink::NGBoxFragmentPainter::PaintBlockFlowContents()
#20 0x7ff912a93b64 blink::NGBoxFragmentPainter::PaintObject()
#21 0x7ff912a92eaf blink::NGBoxFragmentPainter::PaintInternal()
#22 0x7ff912a92757 blink::NGBoxFragmentPainter::Paint()
#23 0x7ff912a96946 blink::NGBoxFragmentPainter::PaintBlockChild()
#24 0x7ff912a960d9 blink::NGBoxFragmentPainter::PaintBlockChildren()
#25 0x7ff912a93ccd blink::NGBoxFragmentPainter::PaintObject()
#26 0x7ff912a92eaf blink::NGBoxFragmentPainter::PaintInternal()
#27 0x7ff912a92757 blink::NGBoxFragmentPainter::Paint()
#28 0x7ff91261d18e blink::LayoutNGMixin<>::Paint()
#29 0x7ff912adbda1 blink::PaintLayerPainter::PaintFragmentWithPhase()
#30 0x7ff912adb495 blink::PaintLayerPainter::PaintWithPhase()
#31 0x7ff912adb8aa blink::PaintLayerPainter::PaintForegroundPhases()
#32 0x7ff912adac0a blink::PaintLayerPainter::PaintLayerContents()
#33 0x7ff912ada22c blink::PaintLayerPainter::Paint()
#34 0x7ff912adb5f0 blink::PaintLayerPainter::PaintChildren()
#35 0x7ff912adacbf blink::PaintLayerPainter::PaintLayerContents()
#36 0x7ff912ada22c blink::PaintLayerPainter::Paint()
#37 0x7ff912adb5f0 blink::PaintLayerPainter::PaintChildren()
#38 0x7ff912adacbf blink::PaintLayerPainter::PaintLayerContents()
#39 0x7ff912ada22c blink::PaintLayerPainter::Paint()
#40 0x7ff912adb5f0 blink::PaintLayerPainter::PaintChildren()
#41 0x7ff912adacbf blink::PaintLayerPainter::PaintLayerContents()
#42 0x7ff912ada22c blink::PaintLayerPainter::Paint()
#43 0x7ff912a5a3c7 blink::FramePainter::Paint()
#44 0x7ff9119cf6f0 blink::LocalFrameView::PaintFrame()
```


```C++
#0 0x7f934abe938f base::debug::CollectStackTrace()
#1 0x7f934a93782a base::debug::StackTrace::StackTrace()
#2 0x7f934a9377e5 base::debug::StackTrace::StackTrace()
#3 0x7f934abe8e5c base::debug::(anonymous namespace)::StackDumpSignalHandler()
#4 0x7f930be3ea00 (/usr/lib64/libc.so.6+0x3e9ff)
#5 0x7f930be28899 __GI_abort
#6 0x7f93184c2ab6 blink::HarfBuzzShaper::Shape()
#7 0x7f931f992bcc blink::NGInlineItemSegments::ShapeText()
#8 0x7f931f9c85fa blink::(anonymous namespace)::ReusingTextShaper::Reshape()
#9 0x7f931f9c5e9c blink::(anonymous namespace)::ReusingTextShaper::Shape()
#10 0x7f931f9c4d23 blink::NGInlineNode::ShapeText()
#11 0x7f931f9c03db blink::NGInlineNode::ShapeTextIncludingFirstLine()
#12 0x7f931f9c0243 blink::NGInlineNode::ShapeTextOrDefer()
#13 0x7f931f9c725c blink::NGInlineNode::ComputeMinMaxSizes()
#14 0x7f931fa557a2 blink::NGBlockLayoutAlgorithm::ComputeMinMaxSizes()
#15 0x7f931fa72cd9 _ZZN5blink12_GLOBAL__N_131ComputeMinMaxSizesWithAlgorithmERKNS_23NGLayoutAlgorithmParamsERKNS_21MinMaxSizesFloatInputEENKUlPNS_27NGLayoutAlgorithmOperationsEE_clES8_
#16 0x7f931fa72c79 _ZN5blink12_GLOBAL__N_121CreateAlgorithmAndRunINS_22NGBlockLayoutAlgorithmEZNS0_31ComputeMinMaxSizesWithAlgorithmERKNS_23NGLayoutAlgorithmParamsERKNS_21MinMaxSizesFloatInputEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS5_RKT0_
#17 0x7f931fa72301 _ZN5blink12_GLOBAL__N_124DetermineAlgorithmAndRunIZNS0_31ComputeMinMaxSizesWithAlgorithmERKNS_23NGLayoutAlgorithmParamsERKNS_21MinMaxSizesFloatInputEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS4_RKT_
#18 0x7f931fa6e00e blink::(anonymous namespace)::ComputeMinMaxSizesWithAlgorithm()
#19 0x7f931fa6d88a blink::NGBlockNode::ComputeMinMaxSizes()
#20 0x7f931fabb273 blink::(anonymous namespace)::ComputeInlineSizeForFragmentInternal()::$_6::operator()()
#21 0x7f931fabaed0 blink::ResolveMainInlineLength<>()
#22 0x7f931fab50ad blink::(anonymous namespace)::ComputeInlineSizeForFragmentInternal()
#23 0x7f931fab4de3 blink::ComputeInlineSizeForFragment()
#24 0x7f931fab99a8 blink::CalculateInitialFragmentGeometry()
#25 0x7f931fa68f3b blink::NGBlockNode::Layout()
#26 0x7f931fa706c6 blink::NGBlockNode::LayoutAtomicInline()
#27 0x7f931f9e2ecd blink::NGLineBreaker::HandleAtomicInline()
#28 0x7f931f9df5b3 blink::NGLineBreaker::BreakLine()
#29 0x7f931f9dee0e blink::NGLineBreaker::NextLine()
#30 0x7f931f9b2e47 blink::NGInlineLayoutAlgorithm::Layout()
#31 0x7f931f9c6b89 blink::NGInlineNode::Layout()
#32 0x7f931fa5f897 blink::(anonymous namespace)::LayoutInflow()
#33 0x7f931fa5f743 blink::NGBlockLayoutAlgorithm::HandleInflow()
#34 0x7f931fa64bdf blink::NGBlockLayoutAlgorithm::Layout()
#35 0x7f931fa56878 blink::NGBlockLayoutAlgorithm::LayoutWithItemsBuilder()
#36 0x7f931fa565af blink::NGBlockLayoutAlgorithm::LayoutWithInlineChildLayoutContext()
#37 0x7f931fa5630c blink::NGBlockLayoutAlgorithm::Layout()
#38 0x7f931fa71a11 _ZZN5blink12_GLOBAL__N_119LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEENKUlPNS_27NGLayoutAlgorithmOperationsEE_clES5_
#39 0x7f931fa719b9 _ZN5blink12_GLOBAL__N_121CreateAlgorithmAndRunINS_22NGBlockLayoutAlgorithmEZNS0_19LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS5_RKT0_
#40 0x7f931fa71011 _ZN5blink12_GLOBAL__N_124DetermineAlgorithmAndRunIZNS0_19LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS4_RKT_
#41 0x7f931fa6af27 blink::(anonymous namespace)::LayoutWithAlgorithm()
#42 0x7f931fa69467 blink::NGBlockNode::Layout()
#43 0x7f931fa5f085 blink::(anonymous namespace)::LayoutBlockChild()
#44 0x7f931fa5f8d5 blink::(anonymous namespace)::LayoutInflow()
#45 0x7f931fa5f743 blink::NGBlockLayoutAlgorithm::HandleInflow()
#46 0x7f931fa64bdf blink::NGBlockLayoutAlgorithm::Layout()
#47 0x7f931fa56322 blink::NGBlockLayoutAlgorithm::Layout()
#48 0x7f931fa71a11 _ZZN5blink12_GLOBAL__N_119LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEENKUlPNS_27NGLayoutAlgorithmOperationsEE_clES5_
#49 0x7f931fa719b9 _ZN5blink12_GLOBAL__N_121CreateAlgorithmAndRunINS_22NGBlockLayoutAlgorithmEZNS0_19LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS5_RKT0_
#50 0x7f931fa71011 _ZN5blink12_GLOBAL__N_124DetermineAlgorithmAndRunIZNS0_19LayoutWithAlgorithmERKNS_23NGLayoutAlgorithmParamsEEUlPNS_27NGLayoutAlgorithmOperationsEE_EEvS4_RKT_
#51 0x7f931fa6af27 blink::(anonymous namespace)::LayoutWithAlgorithm()
#52 0x7f931fa69467 blink::NGBlockNode::Layout()
#53 0x7f931fac5f36 blink::NGOutOfFlowLayoutPart::GenerateFragment()
#54 0x7f931fac5a2c blink::NGOutOfFlowLayoutPart::Layout()
#55 0x7f931fabf857 blink::NGOutOfFlowLayoutPart::LayoutOOFNode()
#56 0x7f931fabe15e blink::NGOutOfFlowLayoutPart::LayoutCandidates()
#57 0x7f931fabd74f blink::NGOutOfFlowLayoutPart::Run()
#58 0x7f931fa1e65f blink::LayoutNGMixin<>::UpdateOutOfFlowBlockLayout()
#59 0x7f931fa0d468 blink::LayoutNGBlockFlowMixin<>::UpdateNGBlockLayout()
#60 0x7f931fa0c31d blink::LayoutNGBlockFlow::UpdateBlockLayout()
#61 0x7f931f652ced blink::LayoutBlock::UpdateLayout()
#62 0x7f931f654cab blink::LayoutBlock::LayoutPositionedObject()
#63 0x7f931f6548d6 blink::LayoutBlock::LayoutPositionedObjects()
#64 0x7f931f67af87 blink::LayoutBlockFlow::UpdateBlockLayout()
#65 0x7f931f84194c blink::LayoutView::UpdateBlockLayout()
#66 0x7f931f652ced blink::LayoutBlock::UpdateLayout()
#67 0x7f931f841c48 blink::LayoutView::UpdateLayout()
#68 0x7f931edc17cd blink::LocalFrameView::PerformLayout()
#69 0x7f931edc28c0 blink::LocalFrameView::UpdateLayout()
#70 0x7f931edd000a blink::LocalFrameView::UpdateStyleAndLayoutInternal()
#71 0x7f931edc5e05 blink::LocalFrameView::UpdateStyleAndLayout()
#72 0x7f931edcd51a blink::LocalFrameView::UpdateStyleAndLayoutIfNeededRecursive()
#73 0x7f931edcba96 blink::LocalFrameView::RunStyleAndLayoutLifecyclePhases()
#74 0x7f931edcb060 blink::LocalFrameView::UpdateLifecyclePhasesInternal()
#75 0x7f931edca140 blink::LocalFrameView::UpdateLifecyclePhases()
#76 0x7f931edca738 blink::LocalFrameView::UpdateLifecycleToLayoutClean()
#77 0x7f931fdbde47 blink::PageAnimator::UpdateLifecycleToLayoutClean()
#78 0x7f931fdc4688 blink::PageWidgetDelegate::UpdateLifecycle()
#79 0x7f931eeda144 blink::WebFrameWidgetImpl::UpdateLifecycle()
#80 0x7f9320e2d6ba blink::WebViewImpl::ResizeViewWhileAnchored()
#81 0x7f9320e2dc90 blink::WebViewImpl::ResizeWithBrowserControls()
#82 0x7f931eedb2a1 blink::WebFrameWidgetImpl::ApplyVisualPropertiesSizing()
#83 0x7f931eeda934 blink::WebFrameWidgetImpl::UpdateVisualProperties()
#84 0x7f9318ac45ff blink::WidgetBase::UpdateVisualProperties()
#85 0x7f931b0cafd0 blink::mojom::blink::WidgetStubDispatch::Accept()
#86 0x7f9318ad1410 blink::mojom::blink::WidgetStub<>::Accept()
#87 0x7f934ae25167 mojo::InterfaceEndpointClient::HandleValidatedMessage()
#88 0x7f934ae24999 mojo::InterfaceEndpointClient::HandleIncomingMessageThunk::Accept()
#89 0x7f934ae3d20b mojo::MessageDispatcher::Accept()
#90 0x7f934ae26caf mojo::InterfaceEndpointClient::HandleIncomingMessage()
#91 0x7f9348ba39fa IPC::(anonymous namespace)::ChannelAssociatedGroupController::AcceptOnEndpointThread()
#92 0x7f9348b9890e base::internal::FunctorTraits<>::Invoke<>()
#93 0x7f9348b987b6 base::internal::InvokeHelper<>::MakeItSo<>()
#94 0x7f9348b98723 _ZN4base8internal7InvokerINS0_9BindStateIMN3IPC12_GLOBAL__N_132ChannelAssociatedGroupControllerEFvN4mojo7MessageEEJ13scoped_refptrIS5_ES7_EEEFvvEE7RunImplIS9_NSt4__Cr5tupleIJSB_S7_EEEJLm0ELm1EEEEvOT_OT0_NSG_16integer_sequenceImJXspT1_EEEE
#95 0x7f9348b9862c base::internal::Invoker<>::RunOnce()
#96 0x7f934a8e2d91 _ZNO4base12OnceCallbackIFvvEE3RunEv
#97 0x7f934aad3c76 base::TaskAnnotator::RunTaskImpl()
#98 0x7f934ab2b730 base::TaskAnnotator::RunTask<>()
#99 0x7f934ab2b4d4 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWorkImpl()
#100 0x7f934ab2abce base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#101 0x7f934ab2b6b0 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::DoWork()
#102 0x7f934a9b1e8f base::MessagePumpDefault::Run()
#103 0x7f934ab2bc32 base::sequence_manager::internal::ThreadControllerWithMessagePumpImpl::Run()
#104 0x7f934aa63ef9 base::RunLoop::Run()
#105 0x7f93437d3f15 content::RendererMain()
#106 0x7f9343cfc009 content::RunZygote()
#107 0x7f9343cfc8e6 content::RunOtherNamedProcessTypeMain()
#108 0x7f9343cfdb17 content::ContentMainRunnerImpl::Run()
#109 0x7f9343cfa3ad content::RunContentProcess()
#110 0x7f9343cfad2a content::ContentMain()
#111 0x56052315c82a ChromeMain
#112 0x56052315c652 main
#113 0x7f930be29510 __libc_start_call_main
#114 0x7f930be295c9 __libc_start_main_alias_2
#115 0x56052315c56a _start
  r8: 0000000000000000  r9: 0000000000000000 r10: 0000000000000008 r11: 0000000000000246
 r12: 0000000000000000 r13: 00007ffd7c1a69f0 r14: 0000000000000000 r15: 00007f934aec2000
  di: 0000000000000001  si: 0000000000000001  bp: 00007f92f81a64c0  bx: 00007f930bff9e90
  dx: 0000000000000000  ax: 0000000000000000  cx: 00007f930be8ebec  sp: 00007ffd7c1991b0
  ip: 00007f930be28899 efl: 0000000000010246 cgf: 002b000000000033 erf: 0000000000000000
 trp: 000000000000000d msk: 0000000000000000 cr2: 0000000000000000
```






Shape先于DrawText