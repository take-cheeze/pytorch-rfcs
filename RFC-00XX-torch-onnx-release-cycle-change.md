

<details>
<summary>Instructions - click to expand</summary>

- Fork the rfcs repo: https://github.com/pytorch/rfcs
- Copy `RFC-0000-template.md` to `RFC-00xx-my-feature.md`, or write your own open-ended proposal. Put care into the details.
- Submit a pull request titled `RFC-00xx-my-feature`. 
    - Assign the `draft` label while composing the RFC. You may find it easier to use a WYSIWYG editor (like Google Docs) when working with a few close collaborators; feel free to use whatever platform you like. Ideally this document is publicly visible and is linked to from the PR.
    - When opening the RFC for general discussion, copy your document into the `RFC-00xx-my-feature.md` file on the PR and assign the `commenting` label.
- Build consensus for your proposal, integrate feedback and revise it as needed, and summarize the outcome of the discussion via a [resolution template](https://github.com/pytorch/rfcs/blob/rfc-process/RFC-0000-template.md#resolution).
    - If the RFC is idle here (no activity for 2 weeks), assign the label `stalled` to the PR.
- Once the discussion has settled, assign a new label based on the level of support:
    - `accepted` if a decision has been made in the RFC
    - `draft` if the author needs to rework the RFC’s proposal
    - `shelved` if there are no plans to move ahead with the current RFC’s proposal. We want neither to think about evaluating the proposal
nor about implementing the described feature until some time in the future.
- A state of `accepted` means that the core team has agreed in principle to the proposal, and it is ready for implementation. 
- The author (or any interested developer) should next open a tracking issue on Github corresponding to the RFC.
    - This tracking issue should contain the implementation next steps. Link to this tracking issue on the RFC (in the Resolution > Next Steps section)
- Once all relevant PRs are merged, the RFC’s status label can be finally updated to `closed`.

</details>





# [Title]

**Authors:**
* @take-cheeze (PFN)
* @nickname 


## **Summary**
A short paragraph or bullet list that quickly explains what you're trying to do.

- Move `torch.onnx` module to independent repository from pytorch/pytorch for faster release cycle
- Re-implement `torch.onnx` without C++ code to make torch version independent


## **Motivation**
What motivates this proposal and why is it important?
How should users and developers think about this feature, how would it impact the way PyTorch is used?
Explain impact and value of this feature

Recent updates made to `torch.onnx` is breaking some non-official extensions made to export ONNX models from torch models. For example [change in 1.13 which removed `torch.onnx.symbolic_registry`](https://github.com/pytorch/pytorch/pull/84382) without deperacation notification.
Module structure of `torch.onnx` is quite difficult to debug/modify since the code is mixed up with C++ parts and Python parts which makes language switching difficult for developers. For a proof of concept I've wrote `torch.onnx` like onnx exporter purely written using `torch.jit` module features which is less than a thousand lines of code called [`pfto`](https://github.com/pfnet/pytorch-pfn-extras/blob/master/pytorch_pfn_extras/onnx/pfto_exporter/export.py) (just short name for PFN torch.onnx). Also `torch.onnx` has its own `libonnx` built inside for ONNX shape inference, that makes support hard for custom domain ONNX operator extensions caused by custom operator registeration difficulty.
By having same release cycle with PyTorch is bothering community to evolve ONNX use because PyTorch and ONNXRuntime have 3-month release cycle, ONNX have 6-month release cycle, it may end up to wait a year for new functionality to be available. By using custom build `libonnx` and custom `torch.onnx` symbolic function could reduce this pain but it still makes an extra cost for maintaining non-official community stuff. Changes made in `torch.onnx` isn't mostly specific to PyTorch update itself so it doesn't have to be released in same time.


## **Proposed Implementation**
This is the bulk of the RFC. Explain the design in enough detail for somebody familiar with PyTorch to understand, and for somebody familiar with the implementation to implement. 
This should get into specifics and corner-cases, and include examples of how the feature is used, and how it will interact with other features. Any new terminology should be defined here.
Consider:
*   using examples and diagrams to help illustrate your ideas.
*   including code examples, if you're proposing an interface or system contract.
*   linking to project briefs or wireframes that are relevant.

- Remove `torch.onnx` from `pytorch/pytorch` repository in 2 or 3 releases of PyTorch and have a monthly or weekly release cycle of new indepenent `torch.onnx` module
- Name change to `torch_onnx` or somewhat better name to keep module independent
- Extend support for latest few release of PyTorch for long term support of ONNX export
- Remove C++ codes or reduce it to `torch.jit` pass since some pass maybe difficult to implement only with Python
  - Extending PyTorch is lot more easier than building `torch.onnx` as a part of PyTorch because it won't cause rebuilds other relevant PyTorch modules
- Catch up test coverages to current `torch.onnx` functionallity


## **Metrics **
What are the main metrics to measure the value of this feature? 

Adding new symbolic function or fixing bugs for already released `torch.onnx` should be ideally fixed in 1-month or 1-week.
Also new symbolic functions for newly released ONNX opset should be supported before next PyTorch release.


## **Drawbacks**
Are there any reasons why we should not do this? Here we aim to evaluate risk and check ourselves.

Please consider:
* is it a breaking change?
* Impact on UX
* implementation cost, both in terms of code size and complexity
* integration of this feature with other existing and planned features

ONNX model exporting would be slower than implementing using C++, though we don't have profiling of exporter. We should add profiling feature in new exporter to keep track of peformance.


## **Alternatives**
What other designs have been considered? What is the impact of not doing this?


## **Prior Art**
Discuss prior art (both good and bad) in relation to this proposal:
* Does this feature exist in other libraries? What experience has their community had?
* What lessons can be learned from other implementations of this feature?
* Published papers or great posts that discuss this

- [pfto](https://github.com/pfnet/pytorch-pfn-extras/blob/master/pytorch_pfn_extras/onnx/pfto_exporter/export.py) is also mentioned in motivation section. It's one of torch to ONNX exporter using features of `torch.onnx`. Symbolic functions is one of the heavily used feature from `torch.onnx` since re-implementing each cost more than re-implementing core exporter feature. Converting `aten` operators to `ONNX` opeartor is mostly supported, some primitive operators like `prim::If`, `prim::Constant` is supported.
- [poptorch](https://github.com/graphcore/poptorch) by Graphcore have torchscript to ONNX converter which is wholely implemented in C++ that could be called by Python. Unlike `torch.onnx` they don't need full support of aten operators so it should fit to there use


## **How we teach this**
* What names and terminology work best for these concepts and why? How is this idea best presented?
* Would the acceptance of this proposal mean the PyTorch documentation must be re-organized or altered?
* How should this feature be taught to existing PyTorch users?


## **Unresolved questions**
* What parts of the design do you expect to resolve through the RFC process before this gets merged?
* What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
* What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?


## Resolution
We decided to do it. X% of the engineering team actively approved of this change.

### Level of Support
Choose one of the following:
* 1: Overwhelming positive feedback.
* 2: Positive feedback.
* 3: Majority Acceptance, with conflicting Feedback.
* 4: Acceptance, with Little Feedback.
* 5: Unclear Resolution.
* 6: RFC Rejected.
* 7: RFC Rejected, with Conflicting Feedback.


#### Additional Context
Some people were in favor of it, but some people didn’t want it for project X.


### Next Steps
Will implement it. 


#### Tracking issue
<github issue URL>


#### Exceptions
Not implementing on project X now. Will revisit the decision in 1 year.