Xerces-C Sample Programs - Security Notes
==========================================

IMPORTANT SECURITY INFORMATION
-------------------------------

XXE (XML External Entity) Protection
-------------------------------------
As of the current version, Xerces-C has secure defaults enabled to protect
against XXE attacks:

1. External DTD loading is DISABLED by default
2. Default entity resolution is DISABLED by default

These samples demonstrate various parser features, but when processing
XML from untrusted sources in production applications, you should:

1. Keep the secure defaults (do NOT enable setLoadExternalDTD unless necessary)
2. Keep entity resolution disabled (do NOT set setDisableDefaultEntityResolution to false)
3. Use a SecurityManager to limit entity expansion
4. Consider disabling DOCTYPE processing entirely (setDisallowDoctype(true))
5. If you need to process external entities, implement a restrictive EntityResolver
   that whitelists specific trusted resources (see Redirect sample for example)

Secure Parser Example
---------------------
Here's how to configure a parser securely:

    SAXParser parser;

    // Secure defaults are already set:
    // - setLoadExternalDTD(false)
    // - setDisableDefaultEntityResolution(true)

    // Add entity expansion protection
    SecurityManager* secMgr = new SecurityManager();
    secMgr->setEntityExpansionLimit(10000);
    parser.setSecurityManager(secMgr);

    // Optional: Disable DOCTYPE if not needed
    // parser.setDisallowDoctype(true);

    // If you need external entities, use a restrictive EntityResolver
    // parser.setXMLEntityResolver(mySecureResolver);

    parser.parse(xmlFile);
    delete secMgr;

Warning About Enabling External Entity Processing
-------------------------------------------------
Some samples may demonstrate features that require external entity processing.
NEVER enable these features when processing untrusted XML without proper
security controls:

❌ DANGEROUS (only if processing untrusted XML):
    parser.setLoadExternalDTD(true);
    parser.setDisableDefaultEntityResolution(false);

✓ SAFE:
    // Use secure defaults (already set)
    // Implement restrictive EntityResolver if needed
    parser.setXMLEntityResolver(mySecureResolver);

For More Information
--------------------
See doc/secadv/XXE-PREVENTION.txt for detailed security guidance.

Last Updated: December 2025
