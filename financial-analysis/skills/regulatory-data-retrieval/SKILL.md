# Regulatory Data Retrieval

## Overview
Retrieve regulatory data, filings, and compliance information from European (BAFIN, AMF, ESMA, ECB), UK (FCA, BoE), Japanese (FSA), and Chinese (CSRC) regulatory authorities.

## Use Cases
- **EU AI Act / GDPR Compliance**: Access regulatory guidance, technical standards, and enforcement decisions
- **Financial Services Regulation**: Monitor regulatory changes across multiple jurisdictions
- **Cross-border Compliance**: Track requirements for multinational operations
- **Risk Assessment**: Identify regulatory risks in target markets
- **Due Diligence**: Verify regulatory status of entities and products

## Data Sources

### Tier 1: European Union Regulatory Authorities

#### ESMA (European Securities and Markets Authority)
- **MCP Server**: `esma-esef` (already configured in .mcp.json)
- **Coverage**: EU-wide securities regulation, ESEF filings, MiFID II data
- **Key Datasets**:
  - FIRDS (Financial Instruments Reference Data System)
  - ESEF corporate reporting (iXBRL format)
  - Alternative Investment Funds register
  - Credit Rating Agencies register
- **Access**: Public REST API, no authentication
- **Documentation**: https://registers.esma.europa.eu/

#### ECB (European Central Bank) - Regulatory Data
- **MCP Server**: `ecb-sdw` (already configured in .mcp.json)
- **Coverage**: Banking supervision, monetary policy decisions, regulatory standards
- **Key Datasets**:
  - Single Supervisory Mechanism (SSM) decisions
  - Banking regulation statistics
  - Macroprudential policy instruments
- **Access**: Public Statistical Data Warehouse API
- **Documentation**: https://www.bankingsupervision.europa.eu/

#### BAFIN (German Federal Financial Supervisory Authority)
- **Setup Required**: HTTP Request node in N8N
- **Coverage**: German financial services regulation, BaFin announcements
- **Key Data**:
  - Company database (authorized institutions)
  - Administrative measures and sanctions
  - Prospectuses and notifications
- **Base URL**: https://www.bafin.de/SiteGlobals/Forms/Suche/EN/Expertensuche_Formular.html
- **Access**: Public website (requires web scraping or manual download)
- **Note**: No official API; data available via search interface and PDF downloads

#### AMF (French Autorité des Marchés Financiers)
- **Setup Required**: HTTP Request node in N8N
- **Coverage**: French securities regulation, enforcement actions
- **Key Data**:
  - Regulated entities database
  - Sanctions decisions
  - Product approvals (UCITS, AIFs)
  - Prospectus filings
- **Base URL**: https://www.amf-france.org/en
- **Access**: Public website with search functionality
- **API**: Limited public API for certain registers
- **Documentation**: https://www.amf-france.org/en/professionals/registers-databases

### Tier 2: United Kingdom Regulatory Authorities

#### FCA (Financial Conduct Authority)
- **Setup Required**: HTTP Request node in N8N
- **Coverage**: UK financial services regulation post-Brexit
- **Key Data**:
  - Financial Services Register (authorized firms)
  - Product Sales Database
  - Enforcement actions and fines
  - Regulatory decisions and policy statements
- **Base URL**: https://www.fca.org.uk/
- **API**: Financial Services Register API available
- **Access**: Mixed (some datasets via API, others web-based)
- **Documentation**: https://register.fca.org.uk/

#### BoE (Bank of England) - Regulatory Function
- **MCP Server**: `boe-api` (already configured in .mcp.json for economic data)
- **Coverage**: Prudential Regulation Authority (PRA) supervisory data
- **Key Data**:
  - PRA-regulated firms
  - Regulatory capital requirements
  - Stress test results
- **Access**: Public website and statistical releases
- **Documentation**: https://www.bankofengland.co.uk/prudential-regulation

### Tier 3: Asian Regulatory Authorities

#### FSA Japan (Financial Services Agency)
- **Setup Required**: HTTP Request node in N8N
- **Coverage**: Japanese financial services regulation
- **Key Data**:
  - Licensed financial institutions
  - Administrative actions
  - Regulatory guidelines (often Japanese language)
- **Base URL**: https://www.fsa.go.jp/en/
- **Access**: Public website (limited English content)
- **Note**: Primary documentation in Japanese; English summaries available

#### CSRC China (China Securities Regulatory Commission)
- **Setup Required**: HTTP Request node in N8N  
- **Coverage**: Chinese securities and capital markets regulation
- **Key Data**:
  - Listed companies disclosures
  - IPO approvals
  - Enforcement actions
- **Base URL**: http://www.csrc.gov.cn/csrc_en/
- **Access**: Public website (limited English content)
- **Note**: Great Firewall considerations for direct API access from EU/US
- **Alternative**: Use Hong Kong SFC (Securities and Futures Commission) for more accessible data

## Implementation in N8N

### Workflow Structure
1. **Input Node**: Specify jurisdiction, entity name/ID, or search criteria
2. **HTTP Request Nodes**: Configure for each regulatory authority
3. **Data Parsing**: 
   - JSON parsing for API responses
   - HTML parsing for web scraping (use Cheerio or HTML Extract)
   - PDF extraction for document-heavy sources (use pdf-parse or external service)
4. **Data Normalization**: Standardize formats across jurisdictions
5. **Output Node**: Return structured JSON with regulatory data

### Authentication & Access
- **EU/UK Sources**: Mostly NO_AUTH (public access)
- **Rate Limiting**: Implement delays between requests (1-2 seconds)
- **Headers Required**: 
  - User-Agent (identify your application)
  - Accept-Language (for multilingual sites)

### Quality Assurance
- **Data Freshness**: Add timestamp to all retrieved data
- **Source Attribution**: Always cite the regulatory authority and specific database
- **Validation**: Cross-reference entity identifiers (LEI, registration numbers)
- **Error Handling**: Gracefully handle API downtime or format changes

## Compliance Considerations

### EU AI Act Relevance
- Track AI system registrations (when EU AI Act database goes live)
- Monitor conformity assessments and certifications
- Access harmonized standards published by ESMA/ECB

### GDPR Compliance
- **Personal Data**: Avoid collecting personal data of individuals from regulatory registers
- **Purpose Limitation**: Only retrieve data necessary for specified compliance/analysis purposes
- **Data Minimization**: Do not store entire regulatory databases; fetch on-demand
- **Legitimate Interest**: Document business justification for accessing regulatory data

## Example Queries

### Search for Authorized Entity
```
Input: Entity name "Deutsche Bank AG"
Steps:
1. Query BAFIN company database
2. Query ECB SSM register
3. Query ESMA registers (if applicable for securities activities)
Output: Consolidated authorization status across EU regulators
```

### Monitor Regulatory Changes
```
Input: Jurisdiction "France", Topic "Crypto-assets"
Steps:
1. Fetch recent AMF announcements and positions
2. Check ESMA guidelines on MiCA implementation
3. Parse ECB opinions on digital finance
Output: Timeline of regulatory developments with source links
```

### Cross-Border Compliance Check
```
Input: Entity "FinTech Startup GmbH", Target Markets ["FR", "UK", "JP"]
Steps:
1. Check BAFIN authorization (Germany)
2. Verify AMF notification requirements (France)
3. Query FCA equivalence rules (UK)
4. Check FSA licensing requirements (Japan)
Output: Compliance roadmap with jurisdiction-specific requirements
```

## Limitations & Workarounds

### No Official APIs
Many regulators (BAFIN, AMF, FSA Japan) lack comprehensive APIs:
- **Workaround**: Use structured web scraping with N8N HTTP Request + HTML parsing
- **Alternative**: Manual download + upload to N8N file trigger workflow

### Language Barriers
Japanese and Chinese sources have limited English content:
- **Workaround**: Integrate translation API (Google Translate, DeepL) in N8N workflow
- **Alternative**: Focus on English-language summaries and international filings

### Data Format Inconsistency
Each regulator uses different structures:
- **Solution**: Build normalization layer in N8N with Function nodes
- **Schema**: Define common output format (JSON schema) for all regulatory data

## Integration with Other Skills

This skill complements:
- **Data Sourcing**: Use regulatory data to validate market data quality
- **Competitive Analysis**: Assess competitors' regulatory compliance status  
- **Risk Assessment**: Identify regulatory risks in investment decisions
- **Due Diligence**: Verify regulatory standing of acquisition targets

## Maintenance

- **Monthly**: Check for API endpoint changes or new data sources
- **Quarterly**: Review and update regulatory authority list (new agencies, restructuring)
- **Annual**: Validate compliance with data protection regulations for storage/processing

## References

- EU AI Act Official Text: https://eur-lex.europa.eu/eli/reg/2024/1689/oj
- GDPR: https://gdpr-info.eu/
- ESMA Registers: https://registers.esma.europa.eu/
- ECB Banking Supervision: https://www.bankingsupervision.europa.eu/
- BAFIN: https://www.bafin.de/EN/
- AMF: https://www.amf-france.org/en
- FCA: https://www.fca.org.uk/
- FSA Japan: https://www.fsa.go.jp/en/
- CSRC: http://www.csrc.gov.cn/csrc_en/
