# EUDI ARF - Intermediary(중개자) 관련 내용 종합 정리

> 근거 문서: `docs/architecture-and-reference-framework-main.md`
> 요구사항 문서: `docs/annexes/annex-2/annex-2.02-high-level-requirements-by-topic.md`

---

## 1. 정의

**Intermediary(중개자)**는 Relying Party의 특수한 유형이다. [European Digital Identity Regulation] Article 5b(10)에 따르면:

> "Relying Party를 대신하여 행동하는 중개자는 Relying Party로 간주되며, **거래 내용에 관한 데이터를 저장해서는 안 된다**."

Intermediary는 Relying Party를 대신하여 Wallet Unit에 연결하고, 필요한 사용자 속성(attributes)을 요청한 뒤, 제시된 속성을 중개된 Relying Party에게 전달하는 역할을 한다.

> 근거: `architecture-and-reference-framework-main.md` Section 3.11.3 (line ~1078)

---

## 2. 역할 요약

| 역할                                  | 설명                                     | 근거                                 |
| ------------------------------------- | ---------------------------------------- | ------------------------------------ |
| **Relying Party (RP) / Intermediary** | Wallet Unit으로부터 속성을 요청하고 수신 | Section 3.11 역할 테이블 (line ~799) |

> 근거: `architecture-and-reference-framework-main.md` Section 3 역할 테이블

---

## 3. 등록(Registration) 관련

- Registrar는 Relying Party가 **intermediary의 서비스를 이용할 의도가 있는지**를 등록한다.
- 이용하는 경우, **어떤 intermediary인지**도 함께 등록된다.
- **Reg_26**: Member State는 등록 시 해당 엔티티가 intermediary로 활동할 의도가 있는지를 고려해야 한다.

> 근거: `architecture-and-reference-framework-main.md` Section 3.17 (line ~1169)
> 근거: `annex-2.02-high-level-requirements-by-topic.md` Reg_26 (line ~691)

---

## 4. 사용자 승인(User Approval) 관련

Relying Party가 intermediary를 사용하는 경우:

- Wallet Unit은 사용자에게 **intermediary의 이름과 고유 식별자**도 함께 표시해야 한다.
- Intermediary 정보는 **access certificate**에 포함된다.
- 중개된 Relying Party 정보는 **presentation request의 확장(extension)** 및 **registration certificate**에 포함된다.
- 이 두 정보가 다르면, Wallet Unit은 해당 요청이 intermediary를 통한 것임을 알 수 있다.

> 근거: `architecture-and-reference-framework-main.md` Section 6.6.3.5.3 (line ~4398)

---

## 5. 데이터 삭제 요청

- Intermediary는 Wallet Unit에서 얻은 데이터를 Relying Party에 전달한 **즉시 삭제**해야 한다.
- 따라서 데이터 삭제 요청은 **항상 Relying Party에게** 보내야 하며, intermediary에게 보내는 것이 아니다.

> 근거: `architecture-and-reference-framework-main.md` Section 6.6.3 관련 (line ~4889)

---

## 6. Intermediary를 통한 PID/attestation 제시 전체 흐름

7단계로 구성된다:

| 단계                       | 내용                                                                                                                                                                                                                                                             |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Intermediary 등록**   | Intermediary가 Registrar에 Relying Party로 등록하고, 자신의 이름과 식별자가 담긴 access certificate를 획득. Registration certificate도 받을 수 있으나 중개 거래에는 사용되지 않음.                                                                               |
| **2. 중개 대상 RP 등록**   | Intermediary가 각 중개 대상 Relying Party를 해당 Member State의 Registrar에 별도 등록. 계약 등 증거를 제공하여 관계를 증명. Registration certificate에 intermediary 관계가 명시됨.                                                                               |
| **3. 속성 요청**           | 중개 대상 RP의 요청에 따라 intermediary가 Wallet Unit에 속성 제시를 요청. **자신의 access certificate** + **중개 대상 RP의 registration certificate**(있는 경우)를 사용. Presentation request에 중개 대상 RP의 이름, 식별자, Registrar URL, 사용 목적 등을 포함. |
| **4. Wallet Unit의 검증**  | 사용자가 RP 정보 확인을 원하면, Wallet Unit은 해당 RP가 실제로 이 intermediary의 서비스를 사용하는지 검증 (registration certificate 확인 또는 Registrar에 온라인 조회). 실패 시 사용자에게 알림.                                                                 |
| **5. Intermediary의 검증** | Intermediary가 PID/attestation의 진위성, 폐기 상태, 기기 바인딩, 사용자 바인딩, 결합 제시 등을 검증 (중개 대상 RP와 합의된 범위 내에서).                                                                                                                         |
| **6. 속성 전달**           | 검증 성공 시, intermediary가 사용자 속성을 중개 대상 Relying Party에 전달. Intermediary↔RP 간 인터페이스 사양은 ARF 범위 밖. 사용자 속성의 종단 간 암호화는 필수가 아님.                                                                                         |
| **7. 즉시 삭제**           | Intermediary는 Wallet Unit에서 얻은 모든 PID/attestation 및 사용자 속성을 **전달 직후 즉시 삭제**. 전달하지 않는 경우에도 검증 완료 즉시 삭제.                                                                                                                   |

> **참고**: 중개 대상 Relying Party는 access certificate가 필요 없다.

> 근거: `architecture-and-reference-framework-main.md` Section 6.6.5 (line ~5043)

---

## 7. Topic 52 - 상세 요구사항

| 요구사항    | 내용                                                                                                                                                              |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RPI_01**  | Intermediary는 Relying Party로 등록하면서 intermediary로 활동할 의사를 표시해야 함. 자체 access certificate 획득. 별도로 자체 RP 자격으로도 등록 가능.            |
| **RPI_03**  | 각 중개 대상 RP를 해당 RP가 소재한 Member State의 Registrar에 등록해야 함. Registration certificate 발급 시 수령.                                                 |
| **RPI_04**  | 중개 대상 RP 등록 시 법적으로 유효한 증거(예: 계약서)를 제공해야 하며, Registrar가 이를 검증.                                                                     |
| **RPI_05**  | 중개 대상 RP가 intermediary에 속성 요청 시 제공해야 할 정보: 이름, 식별자, Registrar URL, 사용 목적 식별자, 사용 목적 설명, 사용할 registration certificate 지정. |
| **RPI_06**  | Intermediary가 Wallet Unit에 제시 요청 시 자신의 access certificate + 중개 대상 RP의 registration certificate(가능한 경우) + RPRC_19a 정보를 포함해야 함.         |
| **RPI_07**  | Wallet Unit은 사용자 승인 요청 시 **intermediary와 중개 대상 RP 양쪽의 이름과 식별자를 모두 표시**해야 함.                                                        |
| **RPI_07a** | 사용자가 RP 정보 확인을 원하면, Wallet Unit은 intermediary↔RP 간 계약 관계가 Registrar에 등록되어 있는지 검증해야 함. 실패 시 사용자에게 알림.                    |
| **RPI_08**  | Intermediary는 검증 성공 후 사용자 속성을 **해당 요청을 한 RP에게만** 전달. 검증 실패 시 전달 금지.                                                               |
| **RPI_09**  | Intermediary는 PID/attestation의 진위성, 폐기 상태, 기기 바인딩, 사용자 바인딩 등을 RP와 합의한 범위 내에서 검증해야 함.                                          |
| **RPI_10**  | Intermediary는 모든 PID/attestation 및 사용자 속성을 전달 **즉시 완전 삭제**. 전달하지 않는 경우에도 검증 완료 즉시 삭제.                                         |

> 근거: `annex-2.02-high-level-requirements-by-topic.md` Topic 52 (line ~1071)

---

## 8. 기타 관련 언급

### Registration Certificate 관련

- **RPRC_08**: Registration certificate의 EU 고유 식별자는 동일 엔티티에 대해 항상 동일해야 하며, intermediary가 보유/제시하는 중개 대상 RP의 certificate도 해당 RP의 식별자를 사용.

> 근거: `annex-2.02-high-level-requirements-by-topic.md` RPRC_08 (line ~993)

### Authentic Source 중개자 (별도 개념)

QTSPAS_03~08에서 언급되는 "designated intermediary"는 RP intermediary와는 **다른 개념**이다. QTSP와 Authentic Source 간의 속성 검증을 중개하는 역할이며, OOTS(Once Only Technical System) 노드가 이 역할의 후보로 언급된다.

> 근거: `annex-2.02-high-level-requirements-by-topic.md` QTSPAS_03~08 (line ~955)

---

## 핵심 원칙 요약

| 원칙                | 설명                                                                  | 주요 근거                              |
| ------------------- | --------------------------------------------------------------------- | -------------------------------------- |
| **법적 지위**       | Intermediary = Relying Party (법적으로 동일 취급)                     | Section 3.11.3, Article 5b(10)         |
| **데이터 비저장**   | 거래 내용 데이터를 저장할 수 없으며, 전달 즉시 삭제 의무              | RPI_10, Section 6.6.5 단계 7           |
| **이중 표시**       | Wallet Unit은 사용자에게 intermediary와 실제 RP 양쪽 정보를 모두 표시 | RPI_07, Section 6.6.3.5.3              |
| **이중 등록**       | Intermediary 자체 등록 + 각 중개 대상 RP의 별도 등록 필요             | RPI_01, RPI_03, Section 6.6.5 단계 1~2 |
| **검증 합의**       | Intermediary가 수행할 검증 범위는 중개 대상 RP와의 합의에 따름        | RPI_09, Section 6.6.5 단계 5           |
| **인터페이스 자율** | Intermediary↔RP 간 인터페이스 사양은 ARF 범위 밖                      | Section 6.6.5 단계 6                   |
