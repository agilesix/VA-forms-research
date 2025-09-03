flowchart TD
    Start([User goes to VA.gov]) --> Auth[User Authenticates]
    Auth --> Complete[User completes form]
    Complete --> Sign[User signs form]
    
    Sign --> Parse[VA.gov parses form data<br/>and produces payload]
    
    Parse --> SubProcess1[["📄 Populate PDF with form data"]]
    Parse --> SubProcess2[["📊 Generate metadata JSON:<br/>• veteranFirstName<br/>• veteranLastName<br/>• fileNumber<br/>• zipCode<br/>• Optional: source, docType, businessLine"]]
    
    SubProcess1 --> Upload
    SubProcess2 --> Upload
    
    Upload[VA.gov POSTs to Benefits Intake /uploads<br/>Gets pre-signed URL, PUTs multipart package]
    
    Upload --> Package[["📦 Package includes:<br/>• content PDF<br/>• metadata JSON<br/>• attachment1..N PDFs optional"]]
    
    Package --> BIResponse[Benefits Intake API<br/>responds with confirmation number]
    
    BIResponse --> ConfPage[User sees confirmation number<br/>on confirmation page]
    BIResponse --> EmailUser1[VA.gov sends user email:<br/>submission in progress<br/>with confirmation number]
    
    BIResponse --> SendEMMS[Benefits Intake API<br/>sends payload to EMMS API]
    
    SendEMMS --> DOMA[EMMS API sends data<br/>to DOMA portal]
    
    DOMA --> DataDim[PDF sent to Data Dimensions<br/>govCIO subcontractor]
    
    DataDim --> Convert[["🔄 Data Dimensions:<br/>• Converts to searchable PDF<br/>• Indexes PDF<br/>• Extracts metadata"]]
    
    Convert --> CMP[Data sent to<br/>Central Mail Portal]
    
    CMP --> AutoCheck{Document type<br/>automated?}
    
    AutoCheck -->|Yes| IBM[IBM Mail Automation Service<br/>OCR/ICR + RPA]
    
    IBM --> IBMProcess[["🤖 IBM Process:<br/>• Ensures completeness<br/>• Searches VBMS for Veteran<br/>• Types data into CMP notes<br/>• Types data into VBMS"]]
    
    IBMProcess --> UploadVBMS1[Uploads form to VBMS<br/>and establishes claim]
    
    AutoCheck -->|No| Manual[Claims assistants<br/>manually enter information]
    
    Manual --> UploadVBMS2[Upload files to eFolder<br/>in VBMS]
    
    UploadVBMS1 --> Poll
    UploadVBMS2 --> Poll
    
    Poll[Lighthouse polls EMMS API<br/>for submission status]
    
    Poll --> Status[["📊 Possible Statuses:<br/>• Pending<br/>• Uploaded<br/>• Received<br/>• Processing<br/>• Success<br/>• VBMS<br/>• Error<br/>• Expired"]]
    
    Status --> StatusCheck{Status reached<br/>VBMS?}
    
    StatusCheck -->|Yes| EmailSuccess[User receives email:<br/>Form reached system of record]
    
    StatusCheck -->|Error/Expired| EmailFail[User receives email:<br/>Action needed with instructions]
    
    EmailSuccess --> Process[Claims processors<br/>process claims in VBMS]
    EmailFail --> End1([End - User action required])
    Process --> End2([End - Claim processing])
    
    style Start fill:#e1f5e1
    style End1 fill:#ffe1e1
    style End2 fill:#e1f5e1
    style AutoCheck fill:#fff4e1
    style StatusCheck fill:#fff4e1
    style SubProcess1 fill:#e1e5ff
    style SubProcess2 fill:#e1e5ff
    style Package fill:#e1e5ff
    style Convert fill:#e1e5ff
    style IBMProcess fill:#e1e5ff
    style Status fill:#e1e5ff
