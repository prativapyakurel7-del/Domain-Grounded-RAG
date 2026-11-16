
Domain-Grounded RAG Architecture for Automated EnterpriseAuthor: Prativa Pyakurel (ID: 77744340)Institution: Department of Computer Science, Leeds Beckett UniversityModule: MSc Advanced Computer Science — Research Practice & DissertationOverviewModern enterprises rely on vast repositories of unstructured internal documents—such as company policies, past project reports, technical capabilities, tender specifications, and candidate CVs—to make rapid operational decisions. While public Large Language Models (LLMs) offer strong reasoning capabilities, transmitting confidential corporate records to external cloud APIs creates severe data-leakage risks, regulatory non-compliance (e.g., GDPR), and unverified hallucinations.This project presents a 100% local, air-gapped Retrieval-Augmented Generation (RAG) architecture operating strictly within an organisation-controlled private computing boundary. It transforms internal documents into a secure, access-controlled knowledge base and pairs hybrid retrieval with local open-source LLMs to deliver verified, cited answers—or honestly abstain when supporting evidence is insufficient.Core Enterprise WorkflowsWhile designed as a general-purpose internal enterprise AI co-pilot, the architecture is validated across two primary operational workflows:Automated Procurement & Bid Compliance Analysis: Evaluates incoming client Requests for Proposals (RFPs) and tender criteria against historical company projects, technical capabilities, and resource records to perform gap analysis and draft grounded bid documentation.Talent Acquisition & Explainable ATS Support: Generates role-specific Job Descriptions (JDs) aligned with internal project stacks, evaluates candidate CVs against job requirements with traceable evidence citations, and generates technical screening questions.Internal Knowledge Q&A: Enables employees and managers to query organisational policies, operational guidelines, and project archives with strict role-based permissions.System ArchitectureThe platform enforces a strict Organisation-Controlled Private Computing Boundary, separating offline knowledge ingestion from online authenticated query execution:flowchart TB
    subgraph BOUNDARY ["ORGANISATION-CONTROLLED PRIVATE COMPUTING BOUNDARY"]
        direction TB

        subgraph INGESTION ["A | KNOWLEDGE INGESTION"]
            direction LR
            n1["<b>Document Sources</b><br>PDF • DOCX •<br>Policies • CVs"] --> n2["<b>Parse + Chunk</b><br>Extract • Clean<br>• Split • Tag"]
            n2 --> n3["<b>Local Embeddings</b><br>BGE / Sentence-<br>Transformers"]
            n3 --> n4["<b>Knowledge Indexes</b><br>Vector DB +<br>BM25 Index"]
        end

        meta["<b>Metadata + ACL Tags:</b> department | sensitivity | owner | date | permissions"]

        subgraph QUERY ["B | SECURE QUERY PATH"]
            direction TB
            subgraph ROW1 [" "]
                direction LR
                q1["<b>User UI</b><br>Chat / Upload"] --> q2["<b>API + Identity</b><br>Auth • RBAC<br>• Input Checks"]
                q2 --> q3["<b>Query Router</b><br>Q&A • Bid • HR<br>• Workflow"]
                q3 --> q4["<b>Hybrid Retrieval</b><br>Dense + BM25<br>Top-k Passages"]
            end

            subgraph ROW2 [" "]
                direction LR
                q5["<b>RRF Fusion</b><br>Merge Ranked Lists"] --> q6["<b>Re-ranker</b><br>Cross-Encoder<br>Refines Order"]
                q6 --> q7["<b>Local LLM</b><br>Llama / Mistral<br>(Candidate Models)"]
                q7 --> q8["<b>Grounded Output</b><br>Citations • Check<br>• Abstain if Weak"]
            end

            ROW1 --> ROW2
        end

        sec["<b>SECURITY & OBSERVABILITY:</b> Local Inference • Outbound Controls • Audit Trail • Access Checks"]

        INGESTION ~~~ meta
        meta ~~~ QUERY
        QUERY ~~~ sec
    end

    classDef monoNode fill:#A6A6A6,stroke:#4D4D4D,stroke-width:1.5px,color:#000000,rx:8,ry:8;
    classDef monoBanner fill:#737373,stroke:#333333,stroke-width:1.5px,color:#FFFFFF,rx:6,ry:6;
    classDef monoNote fill:#D9D9D9,stroke:#8C8C8C,stroke-width:1px,color:#000000,rx:6,ry:6;
    classDef subBoundary fill:#F2F2F2,stroke:#A6A6A6,stroke-width:2px,color:#000000;
    classDef invisible fill:none,stroke:none;

    class n1,n2,n3,n4,q1,q2,q3,q4,q5,q6,q7,q8 monoNode;
    class sec monoBanner;
    class meta monoNote;
    class BOUNDARY subBoundary;
    class INGESTION,QUERY,ROW1,ROW2 invisible;

    linkStyle default stroke:#595959,stroke-width:2px;
Architectural ComponentsA | Knowledge Ingestion Pipeline:Document Sources & Parsing: Extracts, cleans, and structurally chunks internal PDFs, DOCX files, policy handbooks, and CVs.Metadata & ACL Tagging: Attaches granular access control attributes (department, sensitivity, owner, date, permissions) to every document chunk prior to indexing.Local Embeddings & Dual Indexing: Encodes chunks locally using BAAI General Embeddings (bge-large-en-v1.5 / sentence-transformers) into a local Vector Database (ChromaDB) alongside a sparse lexical BM25 index.B | Secure Query & Generation Path:Authentication & RBAC: Verifies user identity and enforces Role-Based Access Control metadata filters before any retrieval occurs, ensuring staff cannot query restricted HR or executive documents.Hybrid Retrieval & RRF Fusion: Combines dense semantic search and sparse BM25 keyword matching using Reciprocal Rank Fusion (RRF) to capture both conceptual context and exact domain terminology.Cross-Encoder Re-ranking: Re-scores top-$k$ candidate passages using a local cross-encoder model to surface the highest-relevance evidence.Local LLM & Grounding Guardrails: Runs quantized open-source models (e.g., Llama-3.1 / Mistral) locally inside Docker containers. Outputs include explicit source citations and trigger an abstention guardrail if retrieval confidence falls below threshold.Security & Governance ControlsSecurity LayerMechanismObjectiveNetwork IsolationAir-gapped Docker deployment with outbound egress blocksGuarantees 0% external data leakage to third-party APIsAccess ControlMetadata-driven RBAC filtering (department, permissions)Enforces least-privilege document visibility across user rolesPII & Input GuardrailsLocal entity detection (Microsoft Presidio / spaCy) & prompt validationPrevents sensitive identifier exposure and indirect prompt injectionHallucination ControlMandatory chunk citations + similarity-threshold abstentionPrevents ungrounded generation when corporate records lack evidenceObservabilityImmutable local query and retrieval audit logsProvides full traceability for enterprise compliance and security auditsImplementation Sequenceflowchart LR
    p1["<b>PHASE 1 | KNOWLEDGE BASE:</b> Build a synthetic company corpus; parse, clean, chunk and label documents."]
    p2["<b>PHASE 2 | RETRIEVAL:</b> Create vector + BM25 indexes; benchmark dense, sparse and hybrid retrieval."]
    p3["<b>PHASE 3 | SECURE RAG:</b> Integrate local LLM, citation checks, RBAC and audit controls."]
    p4["<b>PHASE 4 | EVALUATION:</b> Run task scenarios, security tests and comparative analysis."]

    p1 --> p2 --> p3 --> p4

    classDef monoRect fill:#A6A6A6,stroke:#4D4D4D,stroke-width:1.5px,color:#000000,rx:8,ry:8;
    class p1,p2,p3,p4 monoRect;

    linkStyle default stroke:#595959,stroke-width:2px;
Evaluation FrameworkSystem performance is empirically evaluated on a structured synthetic enterprise dataset (corporate policies, historical project portfolios, tender RFPs, and candidate CVs) across three dimensions:Retrieval & Grounding Quality (RAGAS Framework):Faithfulness: Measures whether generated claims are strictly supported by retrieved internal context.Context Precision & Recall: Compares Dense-only, Sparse (BM25)-only, and Hybrid (RRF + Cross-Encoder) retrieval pipelines.Answer Relevance: Evaluates alignment between user intent and final grounded responses.Security & Boundary Compliance:RBAC Isolation Rate: Verifies 100% blocking of unauthorized cross-department document queries.Abstention Accuracy: Measures refusal rates on unanswerable or out-of-scope queries.External Egress Audit: Confirms zero outbound network requests during ingestion and inference.Operational Efficiency:End-to-end query latency and resource utilization under local containerized execution.ReferencesChroma. (n.d.). Chroma documentation. https://docs.trychroma.com/docs/overview/introductionCormack, G. V., Clarke, C. L. A., & Buettcher, S. (2009). Reciprocal rank fusion outperforms Condorcet and individual rank learning methods. Proceedings of the 32nd International ACM SIGIR Conference on Research and Development in Information Retrieval, 758–759.Es, S., James, J., Espinosa-Anke, L., & Schockaert, S. (2023). RAGAs: Automated evaluation of retrieval augmented generation. arXiv preprint arXiv:2309.15217.Gao, Y., Xiong, Y., Gao, X., Jia, K., Pan, J., Bi, Y., Dai, Y., Sun, J., & Wang, H. (2023). Retrieval-augmented generation for large language models: A survey. arXiv preprint arXiv:2312.10997.Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S., & Kiela, D. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. Advances in Neural Information Processing Systems, 33, 9459–9474.Xiao, S., Liu, Z., Zhang, P., & Muennighoff, N. (2023). C-Pack: Packaged resources for general Chinese embeddings. arXiv preprint arXiv:2309.07597.Disclaimer: Conceptual and experimental MSc research prototype. Model and retrieval configurations are evaluated on synthetic enterprise datasets; no production performance or zero-leakage guarantee is claimed in advance.
