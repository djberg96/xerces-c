XXE Security Vulnerability Fixes - Summary
==========================================

Date: December 26, 2025
Severity: HIGH - XXE vulnerabilities can lead to data disclosure, SSRF, and DoS attacks

OVERVIEW
--------
This commit addresses XML External Entity (XXE) vulnerabilities in Xerces-C by
changing the default parser configuration to be secure by default. Previously,
the parser would load external DTDs and resolve external entities by default,
making applications vulnerable to XXE attacks when processing untrusted XML.

CHANGES MADE
------------

1. Core Security Fixes (XMLScanner.cpp)
   -------------------------------------
   File: src/xercesc/internal/XMLScanner.cpp

   Changed default initialization values in both XMLScanner constructors:
   - fLoadExternalDTD: true → false (SECURITY FIX)
   - fDisableDefaultEntityResolution: false → true (SECURITY FIX)

   Impact: By default, parsers will now:
   - NOT load external DTDs
   - NOT resolve external entities unless a custom EntityResolver is provided

   This prevents XXE attacks by default.

2. Documentation Updates (Parser Headers)
   ----------------------------------------
   Files:
   - src/xercesc/parsers/AbstractDOMParser.hpp
   - src/xercesc/parsers/SAXParser.hpp

   Updated method documentation for:
   - setLoadExternalDTD()
   - getLoadExternalDTD()
   - setDisableDefaultEntityResolution()

   Added security warnings explaining:
   - The new secure defaults
   - Risks of enabling external entity processing
   - When it's safe to enable these features
   - Cross-references between related security methods

3. Security Documentation (program-others.xml)
   --------------------------------------------
   File: doc/program-others.xml

   Updated "Managing Security Vulnerabilities" section to:
   - Document the new secure defaults
   - Explain XXE vulnerability risks
   - Provide secure parser configuration examples
   - Warn against enabling external entity processing with untrusted input

4. Security Advisory (secadv.xml)
   -------------------------------
   File: doc/secadv.xml

   Added new section "Security Improvements in This Release" documenting:
   - The secure defaults change
   - Migration guidance for affected applications
   - Reference to detailed XXE prevention guide

5. XXE Prevention Guide (XXE-PREVENTION.txt)
   ------------------------------------------
   File: doc/secadv/XXE-PREVENTION.txt (NEW)

   Comprehensive security guide including:
   - Overview of XXE attacks
   - Secure defaults explanation
   - Migration guide for existing applications
   - Secure coding practices
   - Example secure parser configurations
   - Testing methodology for XXE vulnerabilities
   - References to OWASP and CWE resources

6. Sample Programs Security Notice (SECURITY-README.txt)
   -----------------------------------------------------
   File: samples/SECURITY-README.txt (NEW)

   Security guidance for sample program users:
   - Explanation of secure defaults
   - Warnings about enabling external entity processing
   - Secure parser configuration examples
   - Reference to detailed security documentation

BACKWARD COMPATIBILITY
----------------------
BREAKING CHANGE: Applications that relied on the previous insecure defaults
will need to explicitly enable external DTD loading if required:

    parser->setLoadExternalDTD(true);  // Only if necessary and input is trusted

Applications using custom EntityResolvers are not affected.

MIGRATION GUIDE
---------------
For applications that need to process external entities:

1. Assess whether external DTD/entity processing is truly necessary
2. If not needed, keep the new secure defaults (no code changes required)
3. If needed:
   a. Verify all XML input sources are trusted
   b. Implement input validation and sanitization
   c. Consider implementing a restrictive EntityResolver
   d. Explicitly enable required features:
      parser->setLoadExternalDTD(true);  // Only if absolutely necessary

TESTING
-------
Recommended testing with XXE payloads (in safe test environment):

1. File disclosure test (should be blocked):
   <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>

2. SSRF test (should be blocked):
   <!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://internal-server/secret">]>

3. Billion laughs test (should be controlled by SecurityManager):
   <!DOCTYPE lolz [<!ENTITY lol "lol"><!ENTITY lol2 "&lol;&lol;...">]>

SECURITY IMPACT
---------------
Before this fix:
- HIGH RISK: Applications were vulnerable to XXE attacks by default
- Attackers could read arbitrary files, perform SSRF, cause DoS

After this fix:
- LOW RISK: Secure defaults prevent XXE attacks
- Only applications that explicitly enable external entity processing
  with untrusted input remain at risk

REFERENCES
----------
- OWASP XXE Prevention: https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html
- CWE-611: https://cwe.mitre.org/data/definitions/611.html
- CVE-2018-1311 (related): Use-after-free vulnerability scanning external DTD

FILES MODIFIED
--------------
1. src/xercesc/internal/XMLScanner.cpp
2. src/xercesc/parsers/AbstractDOMParser.hpp
3. src/xercesc/parsers/SAXParser.hpp
4. doc/program-others.xml
5. doc/secadv.xml

FILES CREATED
-------------
1. doc/secadv/XXE-PREVENTION.txt
2. samples/SECURITY-README.txt

VERIFICATION
------------
All modified files compile without errors.
No build system changes required.
