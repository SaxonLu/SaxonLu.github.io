---
title: Meilisearch Doc
date: 2022-08-08 23:07:16
tags: Meilisearch
---

Meili 是在挪威神話中的神，指"可愛的人"，是托爾的兄弟。
---

----

###Preview

![MeiliSearchPreview.gif]()



**流程圖**
``` mermaid
sequenceDiagram 
    FrontEnd->>API Server: Search Request
    API Server->>+Meilisearch Server:Search Request
    Meilisearch Server->>-API Server:Response
    API Server->>API Server:Format Data
    API Server->>FrontEnd:Response
```