sequenceDiagram
  autonumber
  participant U as Browser/Client
  participant S as Website/Server (www.example.com)
  participant IC as Intermediate CA
  participant RC as Root CA (in trust store)
  participant OCSP as OCSP/CRL Service (CA)
  participant CT as CT Logs (optioneel)
  
  U->>S: 1) ClientHello (SNI=www.example.com, ciphers, ALPN)
  S-->>U: 2) ServerHello (+KeyShare), Certificate (server cert + chain), CertificateVerify, Finished
  
  Note over S,U: Server stuurt zijn certificaatketen mee: <br/>[Server Cert] -> [Intermediate CA Cert] -> (Root CA NIET meegestuurd)
  
  U->>U: 3) Controle domeinna(a)m(en) (CN/SAN = www.example.com)
  U->>U: 4) Controle geldigheid (notBefore/notAfter)
  U->>U: 5) Handtekening-verificatie:<br/>Server Cert getekend door Intermediate?
  U->>U: 6) Verifieer Intermediate getekend door Root?
  U->>RC: 7) Vind Root CA in lokale trust store
  RC-->>U: 8) Root public key (vertrouwd)
  
  rect rgb(245,245,245)
  U->>OCSP: 9) Revocation check (OCSP/CRL) voor Server/Intermediate
  OCSP-->>U: 10) Status: good/revoked/unknown
  U->>CT: 11) (Optioneel) CT-log controle (SCT aanwezig/valide?)
  CT-->>U: 12) Bevestiging CT (optioneel)
  end
  
  alt Alle validaties OK
    U->>S: 13) ClientKeyExchange / Finished (TLS 1.2) <br/> of KeyShare/Finished (TLS 1.3)
    Note over U,S: Sleutelafspraak voltooid → sessiesleutels afgeleid
    S-->>U: 14) Bevestiging / versleutelde data
    U-->>S: 15) Versleuteld HTTP-verkeer (HTTPS)
  else Fout (naam, keten, revocation, verlopen)
    U->>U: Toon foutmelding / blokkeer verbinding
  end
