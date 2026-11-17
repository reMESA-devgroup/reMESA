Reverse Engineering

Our methodology is based on evidence collected through binary analysis, disassembly, register captures, controlled replays, public documentation and experiments on real hardware

Development is divided into sessions, usually documented with dates and times in UTC−3

Each session starts with the results of the previous session, what is already confirmed, what remains unknown and what needs to be implemented or investigated next

We use LLMs extensively throughout development, including code generation, debugging, compiler development, experimental tools and analysis

Information provided by an LLM is not considered hardware documentation, if the AI suggests a register value or an instruction encoding, we need evidence supporting it before considering it confirmed

We don't simply ask an AI to implement a driver and assume that it works

We investigate the original binaries, observe their behavior, collect the relevant data, implement what we understand and compare the results

References, binary hashes, disassembly, command streams, register captures and experimental results are preserved in the repository

Failed experiments are also documented, we don't remove evidence just because the results weren't what we expected

Development is separated into qualification stages, each stage has its own requirements and a successful result in one stage doesn't automatically qualify another

Offline tests are performed before physical tests, including regression tests, memory validation, resource ownership, error handling and expected refusal cases

An offline PASS is not a physical PASS

Most physical tests are performed on my Dell Inspiron Mini 12 with an Intel Atom Z520 and SGX535

Physical validation includes GPU submission, hardware completion, framebuffer readback, independent pixel oracles, memory guards, resource lifetime and recovery

A successful submission doesn't necessarily mean the GPU rendered correctly, which is why we compare the actual output against independently calculated expected results

We also verify that the machine can return to its original driver and normal desktop environment after experimental boots

If a test fails, we investigate the cause instead of removing the validation requirements just to make it pass

Completed stages shouldn't be reopened without new evidence of a problem, and unqualified functionality shouldn't be exposed as supported functionality

We don't consider an OpenGL version fully supported just because a triangle or a specific application renders correctly

Performance measurements also need context, including resolution, actual GPU completion, presentation and whether the operation was performed by the CPU or GPU

Regarding AI and the leaked proprietary SGX source code, we cannot verify the training datasets of the models we use

Our methodology provides experimental traceability, but this doesn't automatically establish the legal provenance of every generated line of code

I'm aware of these limitations and I'm not going to hide the extent of AI usage in this project

The repository contains the available references and evidence behind our technical conclusions, anyone interested can investigate them independently

There are still things we don't understand about the SGX535, including 0x07000345, which remains unresolved as of November 17, 2026

If we don't know something, we document it as unknown

At the end of each development session, we document the results, remaining problems, current qualification status and the next steps

If you have questions about our methodology or the origin of a specific implementation, feel free to open an issue

ty,
Gabriel :)
