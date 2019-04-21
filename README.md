# nestor
Securing Kubernetes Workloads with [Trusted Execution Environments](https://en.wikipedia.org/wiki/Trusted_execution_environment)

Nestor is a collection of tools and reference designs for implementing a TEE aware Kubernetes cluster and microservices.

## Overview
The goal of Nestor is to implement a cloud native infrastructure for collaborative and confidential data analysis deployed to untrusted public clouds. 

Collaborating organizations (academic institutions, government agencies, companies) would like to analyze data over combined datasets while guaranteeing the privacy of their individual data. Participants may not trust - or are compelled by law not to trust - other participants' IT infrastructure or a 3rd party platform host.  Today multi-party collaborative data analysis typically requires directly sharing private data across IT/organizational boundaries, exposing data to leakage, exfiltration, or dissemination.  Further, once data is shared, organizations (academic or commercial) do not always have a secure data analysis infrastructure, nor staff/budgets for proper operations and maintenance, nor a framework for mutually agreed security policies and controls. Organizational policies and privacy regulatory requirements are often prohibitively expensive and complex to implement such that analysts often de-prioritize collaboration - isolating valuable data in silos where it remains un(der)utilized.

Using Asylo and cloud native infrastructure, we propose a TEE-enabled public cloud infrastructure to guarantee that only code inside a trusted enclave has direct access to their unencrypted data, and that there is no way for data to leak from the system.  In such a TEE-enabled infrastructure, each participant will use remote attestation to verify the enclave, generate and securely upload their encryption keys into the enclave, upload their encrypted data, run verified leak-free code that can retain and access sealed intermediate results within the enclave for secure evaluation by all the parties (subject to their agreed access control policies), and finally provide the encrypted results and proof/attestation of execution.

CRI-compliant containers will host workers that will run data analysis jobs. Kubernetes will orchestrate all necessary containers for all job completion.  The code inside the enclave runs Asylo gRPC, verifies the integrity of the code and data, decrypts the data and seals the data as needed, processes the combined data, and computes the final result. We will document several examples to motivate additional real world use cases across several domains:
    
- DEA analysis to benchmark efficiency across competitors at retail service sites
- Long Short-Term Memory RNN for predicting clinical trial targets from multi-center EHR data
- Statistical analysis of employee performance in varying team structures across organizations
- Analysis of user-submitted travel data for local and regional planning agencies, including EV charging projects

We propose delivery of:

- Documentation and code for the enclave-container interface,
- Documentation and code for the attestation, data and key upload, and result download tasks,
- Documentation and code for the container based job queue,
- Documentation and code for the Kubernetes configuration,
- Documentation and code for the data analysis examples,
- Operation of the reference system on Google Cloud made available to researchers for testing and eventual publication of results for a period of 12 months,
- Support for all researchers using the reference system,
- Summary of architectural and design issues, policy guidelines, testing guidelines, bug reports if required, and operations and security guidelines

Additional secondary goals, challenges, and a project roadmap are posted on the Github project. 

## Challenges
- Beyond the primary goal of data privacy guarantees, one would like to guarantee integrity for the whole distributed computation, since the TEE guarantees only integrity of memory regions on individual computers. Nestor would implement a job execution protocol that guarantees the correct and confidential execution of distributed data processing jobs: the computing nodes produce secure summaries of the work they perform, and they aggregate the summaries they receive from their peers. By verifying the final summaries included in the results, the user can check that the cloud provider did not interfere with the computation. At the same time, the cloud provider can freely schedule and balance the computation between the nodes, as long as all data is eventually processed correctly.

- Another challenge is to protect the code running in the isolated memory regions from attacks due to unsafe memory accesses. SGX processors allow this code to access the entire address space of its host process, thus unsafe memory accesses can easily leak data or enable other attacks. Options to prevent this worth investigating include:
    - implementing full memory safety for C/C++ inside Asylo
    - an Asylo-specific compiler that enforces safe pointers reads/writes and call instructions target only safe functions in the isolated region
    - running a verified minimal "no kernel" environment in the enclave (e.g. Asylo-newlib, Asylo with muslc, Asylo with Ariadne)
    - running a verified microkernel in the enclave (e.g. Asylo + seL4 akin to iOS EnclaveOS or Android Trusty/LK)
    - running a verified distributed/lib OS with RPC in the enclave (e.g. Asylo integration with Haven, Graphene)
    - running a verified sandbox container (e.g. Asylo+gVisor)
    - running a verified hypervisor in the enclave (e.g. Asylo+seL4+CAmkES to create something akin to Kata containers)

- Key Management - e.g. Barbican
- Policies (retention policy? result filtering?)
- Preventing exploitation of any side channels induced by data-dependent access patterns (e.g. disk, network, and memory access patterns)
- Preventing Denial of-service and side-channel attacks based on power and timing analysis
- Oblivious computing across participants 

## Threat Model
Actors may control all the hardware in the cloud environment, except the TEE chips: network hardware, disks, and other chips in the motherboards. It is assumed that actors may record, replay, and modify all network traffic. Actors may also read or modify data outside the processor chip using physical probing, direct memory access (DMA), etc. Actors may control all  software in the system: the operating system, orchestrator, container runtime and/or hypervisor. Actors may include privileged malware running in the operating system, container runtime, orchestrator or hypervisor layer, as well as malicious human administrators who may try to access the data by logging into hosts and inspecting disks and memory. We only assume that an attacker is unable to physically open and manipulate the TEE processor chips that run trusted computation tasks.  We assume  all participants receive the result/output of the executed code — limiting/filtering the amount of information released is outside the scope. We consider the source/compiled code to be inspected and/or verified by all participants prior to execution: the code will never intentionally leak plaintext data from enclaves. We assume that all parties agree on the code that gets access to their datasets, and have independently verified its trustworthiness.

## Motivation
Secure Collaborative Computation has many uses in operations research, pharma research/clinical trials, climate/energy initiatives, workforce management/optimization, information security research itself, regulatory and financial oversight - any domain that can benefit from analyzing data across organizations or jurisdictions to produce shared results to the mutual benefit of all participants. 

Today researchers either struggle with prohibitively complex security controls that offer little practical security benefits, or worse simply find workarounds to share data outside of these IT policies and controls. Sadly we have seen many potentially beneficial research projects shelved simply due to the cost, hassle and anxiety over sharing data. 

## Roadmap
- Minimal proof of concept Kubernetes deployment version 0.1 (Q2 2019)
- Minimal job processing framework (Q2 2019)
- Minimal key management and encryption/decryption features (Q3 2019)
- Minimal analysis demonstrations (Q3 2019)
- Initial cohort of research projects and results (Q4'19 - Q1'20)
- Define 1.0 Requirements and recruit additional research projects (Q2 2020)

## Who's Behind Nestor?
Nestor is a group of enthusiastic volunteers who wish to make Confidential Computing in the cloud so convenient and efficient for analysts that they users have no reason to share data insecurely, and can in turn promote secure collaboration in their organizations and industry groups.  We encourage everyone to join!

## How Can I Help?
- At the very least star us on Github to show you care!
- Become a contributor (write code, write tests, help in the formal verification effort, report issues, fix bugs, write documentation, etc): post in the Nestor slack channel for github access
- Do research on Nestor's hosted collaborative cloud: post in the Nestor slack channel for enclave access
- Sponsor Nestor: Nestor runs on volunteer effort but will incur some real world costs for cloud hosting, travel to conferences to present results, sponsoring PhD projects, bug bounties, independent security testing, etc.  We are grateful for any sponsors interested in contributing funds, cloud credits, or services to Nestor.

## FAQs
- Is Nestor a registered non-profit?
    - We would love someone who knows this process to discuss the pros/cons of what structure to create, if any.  Please open an issue if you are an expert in these areas and would like to assist!
- Become a contributor and add some here!
