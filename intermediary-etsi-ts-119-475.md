# ETSI TS 119 475 V1.2.1 - Intermediary 관련 분석

> 근거 문서: `ts_119475v010201p.pdf` (ETSI TS 119 475 V1.2.1, 2026-03)
> 정식 명칭: Electronic Signatures and Trust Infrastructures (ESI); Relying party attributes supporting EUDI Wallet user's authorization decisions

---

## 1. 문서 개요

EUDI Wallet과 상호작용하는 **Wallet-Relying Party(WRP)**의 인증서 프로파일과 정책 요구사항을 정의하는 표준이다. Intermediary 전용 표준은 아니지만, **intermediary가 인증서 체계에서 어떻게 표현되는지**를 기술적으로 정의하는 핵심 표준이다.

주요 정의 내용:

1. **WRPRC**(Registration Certificate)의 정책 및 프로파일 요구사항
2. WRP 정보의 WRPAC/WRPRC 간 매핑 가이드
3. WRPAC과 WRPRC 제공자 간 조정 권고사항

> 근거: Clause 1 Scope (p.8)

---

## 2. 두 가지 인증서 체계

| 인증서                               | 목적                         | 규정 근거                           | Intermediary 관련                          |
| ------------------------------------ | ---------------------------- | ----------------------------------- | ------------------------------------------ |
| **WRPAC** (Access Certificate)       | WRP 인증, 전자서명/전자인감  | Article 7, Annex IV of CIR 2025/848 | Intermediary **자신의** 이름/식별자 포함   |
| **WRPRC** (Registration Certificate) | 사용 목적, 요청 속성, 투명성 | Article 8, Annex V of CIR 2025/848  | 중개 대상 RP 정보 + intermediary 표시 포함 |

> 근거: Table E.1 WRPAC and WRPRC comparison (p.43)

---

## 3. WRPAC에서의 Intermediary

> "WRPACs are digital certificates used by registered WRP, **or an intermediary acting on its behalf WRP**, to authenticate themselves to the EUDIW."

- Intermediary는 **자기 자신의 WRPAC**으로 Wallet과의 연결을 인증한다.
- 하나의 WRP 또는 intermediary가 **여러 개의 WRPAC**을 가질 수 있다 (분산 인스턴스 운영 시).

> 근거: Section 4.3 Wallet-Relying Party Access Certificates (p.14)

---

## 4. WRPRC에서의 Intermediary

Intermediary를 통해 활동하는 WRP의 경우:

- WRPRC의 `sub` 필드는 항상 **최종 Relying Party(중개 대상 RP)**를 식별한다 (intermediary 자체가 아님).
- **WRPRC는 intermediary 전용으로 등록된 WRP에게는 발급되지 않는다.**
- 각 intermediary별로 **별도의 WRPRC가 발급**되어, intermediary와 최종 WRP를 명시적으로 바인딩한다.

> 근거: Table 7 NOTE 2, NOTE 4 (p.23)
> "The subfield always identifies the relying party or the final relying party in case of intermediated transactions. It does not refer to the intermediary itself."
> "WRPRCs are not issued for WRPs registered solely for the purpose of acting as an intermediary."

---

## 5. WRPRC Payload의 Intermediary 필드

### 5.1 필수 요구사항 (GEN-5.2.4-09)

> "If the WRPRC is issued to a WRP acting through an intermediary, the WRPRC **shall** include the field `act.sub` that matches the semantic identifier of the intermediary as specified in clause 5.1."

→ Intermediary를 사용하는 WRP의 WRPRC에는 반드시 intermediary의 식별자가 포함되어야 한다.

> 근거: GEN-5.2.4-09 (p.25)

### 5.2 Optional 필드 (Table 10)

| Attribute        | Field          | Subfield | 설명                                                       |
| ---------------- | -------------- | -------- | ---------------------------------------------------------- |
| usesIntermediary | `intermediary` | -        | WRP가 intermediary를 통해 운영됨을 표시                    |
| usesIntermediary | `intermediary` | `sub`    | Intermediary의 식별자 (WRPAC의 semantic identifier와 매칭) |
| usesIntermediary | `intermediary` | `sname`  | Intermediary의 commonName (WRPAC에 명시된 이름)            |

> NOTE 2: `intermediary` 필드가 있으면 `sub`은 RP를 대신하여 행동하는 intermediary를 식별한다. 중개 거래에서만 사용되며, RP가 직접 통신할 때는 생략된다.

> 근거: Table 10 Mapping of WRPRC optional attributes (p.24)

---

## 6. 데이터 모델에서의 Intermediary (Annex B)

`WalletRelyingParty` 클래스(B.2.1)에 다음 속성이 정의되어 있다:

| Attribute          | Multiplicity | Type                 | Description                                      | CIR 근거   |
| ------------------ | ------------ | -------------------- | ------------------------------------------------ | ---------- |
| `usesIntermediary` | [0..*]       | `WalletRelyingParty` | WRP가 하나 이상의 intermediary에 의존하는지 표시 | Annex I.14 |

> 근거: Annex B, Section B.2.1 Class WalletRelyingParty (p.34)

---

## 7. WRP 식별 속성 비교표 (Annex E)

| WRP attribute    | WRPAC 포함     | WRPRC 포함     | 비고                                                              |
| ---------------- | -------------- | -------------- | ----------------------------------------------------------------- |
| Subject          | Mandatory      | Mandatory      | -                                                                 |
| Friendly name    | Optional       | Optional       | -                                                                 |
| Identifier(s)    | Mandatory (1+) | Mandatory (1+) | 최소 하나는 양쪽 인증서에서 동일                                  |
| URL              | Mandatory      | Optional       | -                                                                 |
| Country code     | Mandatory      | Mandatory      | -                                                                 |
| **Intermediary** | **No**         | **Optional**   | Intermediary의 식별자 포함, WRPAC에 있는 intermediary 정보와 매칭 |

> 근거: Table E.2 WRP identification attributes (p.44)

---

## 8. Registration Certificate 예시 (Annex C)

실제 WRPRC JSON에서 intermediary가 포함된 형태:

```json
{
  "typ": "rc-wrp+jwt",
  "alg": "ES256",
  "x5c": []
}
.
{
  "name": "Example Company",
  "sub": "LEIXG-529900T8BM49AURSDO55",
  "country": "DE",
  "registry_uri": "https://registrar.com",
  "entitlements": ["https://uri.etsi.org/19475/Entitlement/Non_Q_EAA_Provider"],
  "credentials": [ ... ],
  "provides_attestations": [ ... ],
  "intermediary": {
      "sub": "LEIXG-INTERMEDIARY-1234567890",
      "name": "Intermediary Services Ltd."
  }
}
```

핵심 포인트:

- `sub`은 **최종 RP**(Example Company)의 식별자
- `intermediary.sub`은 **intermediary**의 식별자 (WRPAC과 매칭)
- `intermediary.name`은 **intermediary**의 표시 이름

> 근거: Annex C Registration Certificate example (p.39-40)

---

## 9. Intermediary 사용 시 인증서 흐름 (Section 4.5, Annex E.3)

> "When a WRP acts through an intermediary, the intermediary presents its own WRPAC to authenticate the connection with the EUDIW. In parallel, the WRP provides a WRPRC that identifies the final relying party, states the intended purpose, and defines the authorized data access for the transaction."

1. **Intermediary** → 자신의 **WRPAC**으로 Wallet과 연결 인증
2. **중개 대상 RP** → **WRPRC**으로 최종 RP의 신원, 사용 목적, 데이터 접근 권한 명시
3. WRPRC에 `intermediary` 필드가 있어 intermediary와 최종 WRP를 **명시적으로 바인딩**
4. WRPRC가 없는 경우, EUDIW가 national register에서 직접 등록 데이터를 조회

> 근거: Section 4.5 (p.15), Annex E.3 (p.43-44)

---

## 10. ARF 요구사항과의 매핑

| ARF 요구사항                                             | TS 119 475 구현                                                               |
| -------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **RPI_01**: Intermediary가 자체 access certificate 획득  | WRPAC에 intermediary 자신의 이름/식별자 포함 (Section 4.3)                    |
| **RPI_03**: 중개 대상 RP의 registration certificate 수령 | WRPRC가 최종 RP를 `sub`으로 식별 + `intermediary` 필드로 바인딩 (Table 7, 10) |
| **RPI_06**: 요청 시 access cert + registration cert 포함 | WRPAC(intermediary) + WRPRC(중개 대상 RP) 조합 사용 (Section 4.5)             |
| **RPI_07**: Wallet Unit이 양쪽 이름 모두 표시            | WRPAC의 이름(intermediary) + WRPRC의 이름(최종 RP)을 각각 표시                |

---

## 핵심 설계 원칙 요약

| 원칙                                     | 설명                                                                       | 근거                  |
| ---------------------------------------- | -------------------------------------------------------------------------- | --------------------- |
| **인증서 분리**                          | WRPAC = intermediary 자신, WRPRC = 중개 대상 RP → 역할 명확화              | Section 4.3, 4.4, 4.5 |
| **Subject = 최종 RP**                    | WRPRC의 subject는 항상 최종 RP → 실제 데이터 접근 주체를 식별              | Table 7 NOTE 2        |
| **Intermediary별 별도 WRPRC**            | 각 intermediary-RP 조합마다 별도 WRPRC 발급 → 추적 가능성과 정책 집행 보장 | Annex E.3 (p.44)      |
| **Intermediary 전용 WRP에 WRPRC 미발급** | Intermediary 자체가 데이터 접근 목적을 가지지 않음을 기술적으로 보장       | Table 7 NOTE 4        |
