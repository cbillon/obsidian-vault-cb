How do you handle 1M+ API requests daily while keeping response times under 100ms? Here's how we tackled this challenge in building an enterprise integration platform that processes the files for organizations with 50,000+ records.

Let's break down the scale:

- A single enterprise client has 50,000+ employees
    
- Each employee record requires ~20 API calls across internal services
    
    - HRIS data retrieval
        
    - Role history verification
        
    - Other employee-specific information
        
- Multiply this across multiple enterprise clients
    
- Result: Over 1M+ API requests daily, each needing to be fast and reliable
    

In modern HR systems, managing employee benefits at enterprise scale isn't just about moving data - it's about orchestrating a complex dance of internal and external systems. For each benefits processing cycle, our platform needs to:

- Gather employee information across multiple services
    
- Transform massive datasets into vendor-specific file formats
    
- Securely transfer large files (often 100MB+) to benefit providers via SFTP
    
- Process incoming response files containing personalised benefit calculations
    
- Update tens of thousands of employee records in real-time
    

## Core Challenges in Enterprise Benefits Integration

### Scale and Performance

When a single enterprise client with 50,000 employees initiates benefits processing:

- Each employee record requires ~20 internal API calls
    
- Results in 1M+ daily API requests
    
- Generates and processes large files (100MB+ CSVs)
    
- Must maintain sub-100ms response times for real-time operations
    

### Data Consistency

With multiple services involved (HRIS, Role History, etc.), maintaining consistency becomes critical:

- Concurrent employee data updates must be handled
    
- Data must remain consistent across async operations
    
- Partial data availability scenarios need graceful handling
    
- Transaction boundaries must be clearly defined
    

### File Processing

Large-scale file operations bring their own complexities:

- Efficient generation of vendor-specific CSV formats
    
- Memory-efficient processing of 100MB+ files
    
- SFTP transfer reliability for large files
    
- Handling partial file processing scenarios
    

### Security

Processing sensitive employee data requires robust security measures:

- Protection against SSRF (Server Side Request Forgery) attacks
    
    - Vendor files might contain malicious URLs
        
    - Need for strict URL validation
        
    - Prevention of internal network exposure
        
- Secure handling of demographic data
    
- Compliance with data protection regulations
    

### Reliability

At enterprise scale, reliability becomes paramount:

- Network failures during file transfers
    
- Service timeouts and degradations
    
- Vendor system downtimes
    
- Need for robust retry mechanisms
    
- Clear failure recovery paths
    

### Operational Visibility

Managing millions of transactions requires:

- Real-time monitoring capabilities
    
- Quick failure detection
    
- Comprehensive audit trails
    
- SLA compliance tracking
    
- Debugging capabilities across distributed calls
    

## Engineering Solutions: Architecture Patterns & Decisions

Let's explore the key architectural decisions we made when building our benefits integration platform.

### Scale and Performance: Why Batch Processing Won

Our initial approach treated each employee record independently. This seemed logical - process one employee, move to the next. However, with 50,000+ employees, each requiring 20 API calls, this quickly became unsustainable.

Consider this scenario: Processing benefits for a large tech company with offices across five locations. Our initial implementation would make:

- 20 API calls × 50,000 employees = 1M API calls
    
- Each call adding 100ms network latency
    
- Sequential processing taking hours
    

The solution? Batch processing by organizational structure:

- Group employees by location and department
    
- Fetch data in bulk (e.g., all Seattle engineering)
    
- Process related records together
    

This reduced our API calls by 95% and cut processing time from hours to minutes.

### Caching Strategy: The Two-Layer Decision

We faced a critical decision: cache everything or cache selectively? Consider the data patterns:

- Employee demographics change rarely
    
- Role and salary information updates frequently
    
- Benefits eligibility rules remain static
    
- Deduction calculations change monthly
    

This led to our two-layer caching strategy:

1. L1 (In-memory):
    
    - Benefits eligibility rules
        
    - Current pay period data
        
    - Recent calculations
        
2. L2 (Distributed):
    
    - Employee demographics
        
    - Historical calculations
        
    - Department structures
        

### File Processing: Why Streaming Won Over Batching

Early in development, we tried loading entire files into memory. This worked in testing with 100 employees but failed spectacularly with 50,000. A single file with full employee data could exceed 100MB.

Consider the memory implications:

- 50,000 employees × 2KB per record = 100MB
    
- Multiple files being processed simultaneously
    
- Additional memory for processing
    

The streaming pattern emerged as the clear winner:

- Process records as they arrive
    
- Maintain constant memory footprint
    
- Enable parallel processing of chunks
    

## Data Flow and Processing

![[Pasted image 20260416102745.png]]

DBVendorSFTPFileGenServicesProcessorClientDBVendorSFTPFileGenServicesProcessorClientProcess DemographicsCalculate DeductionsProcess Benefits RequestFetch Employee DataEmployee Data (Demographics)Generate Employee Data FileCSV GeneratedUpload FileTransfer FileUpload Deductions FileReceive DeductionsParse Deductions FileParsed DataStore Employee DeductionsConfirmationProcess Complete

## Monitoring & Observability: Tracking Millions of Operations

### Alert Thresholds: Three-Tier Monitoring Strategy

#### P0 (Critical) - Immediate Response Required

- File Transfer Failures:
    
    - 3+ consecutive transfer failures
        
    - File corruption detected
        
    - SFTP connection down >5 minutes
        
- Data Processing:
    
    - Error rate >5% in 5-minute window
        
    - Processing latency >30 minutes
        
    - Database write failures
        

#### P1 (Warning) - Business Hours Response

- Performance Degradation:
    
    - Processing time increased by 50%
        
    - Cache hit rate <80%
        
    - Queue depth >10,000 records
        
- Service Health:
    
    - API latency >200ms (p95)
        
    - Memory usage >85%
        
    - Disk usage >90%
        

#### P2 (Investigation) - Weekly Review

- Trend Analysis:
    
    - 20% increase in processing time
        
    - 10% drop in cache efficiency
        
    - Gradual error rate increase
        
![[Pasted image 20260416102900.png]]
## Lessons Learned: Technical Insights From the Trenches

### What Worked Well

1. Batch Processing Strategy Our decision to process by organizational units rather than individual employees paid off tremendously:

- Reduced API calls by 95%
    
- Better resource utilization
    
- Simpler error handling at batch level
    
- Natural fit for enterprise structure
    

2. Two-Layer Caching The split between in-memory and distributed caching proved crucial:

- Hot data (eligibility rules) stayed ultra-fast
    
- Distributed cache maintained consistency
    
- Memory usage remained predictable
    
- Clear cache invalidation patterns
    

### Unexpected Challenges

1. File Processing Complexity Real example from vendor file:

Issues encountered:

- Commas in values breaking CSV parsing
    
- Mixed date formats in same file
    
- Hidden characters causing validation failures
    
- File size variations (100KB → 100MB)
    

2. State Management Challenges Example race condition we encountered:

```go
// Problematic scenario
func ProcessBenefits(employeeID string) error {
    demo := getDemographic(employeeID)  // T1
    // Meanwhile, employee updates address
    calculateBenefits(demo)             // T2: Using stale data
}

// Solution: Version-based processing
type DemographicData struct {
    Data    Employee
    Version int64
}

func ProcessBenefitsWithVersion(employeeID string) error {
    demo := getDemographicWithVersion(employeeID)
    return processWithOptimisticLock(demo)
}
```

## Conclusion: Building for Enterprise Scale

Building an integration platform that processes benefits for enterprises with 50,000+ employees taught us valuable lessons about scale, reliability, and system design. Let's recap our journey:

### Key Takeaways

1. Scale isn't just about handling large numbers; it's about:

- Smart batching over individual processing
    
- Strategic caching decisions
    
- Efficient resource utilization
    
- Predictable performance patterns
    

2. Reliability at enterprise scale means:

- Robust file processing
    
- Comprehensive monitoring
    
- Well-defined alert thresholds
    
- Clear incident response paths
    

3. The right architecture decisions early on matter:

- Batch processing saved us from millions of API calls
    
- Two-layer caching proved invaluable
    
- Investment in monitoring paid dividends
    
- File processing needed more attention than expected
    

### Moving Forward

As we continue to evolve this platform, our focus remains on:

- Enhanced automation
    
- More sophisticated monitoring
    
- Better developer tooling
    
- Continued performance optimization
    

## Repository Link

[https://github.com/AkshayContributes/load-balancer](https://github.com/AkshayContributes/load-balancer)

The journey from processing a single employee's benefits to handling enterprises with 50,000+ employees has been one of continuous learning and adaptation. The lessons learned here continue to influence our architectural decisions and system design approaches.