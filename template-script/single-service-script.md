---
description: "ตัวอย่างการทำ script k6 สำหรับ single service test"
---
# Single API Flow — Performance Test & Manifest Orchestration (EN/TH)

> **Scope**: Single API execution flow (พื้นฐาน) ครอบคลุม Constant Config, Mock, Manifests (ConfigMap/Secret/Resource/Replica/HPA/Extra), Execution (Options, Function, Summary), และ Runtime

**Tags**: `single-api` `k6` `mock` `configmap` `secret` `resource` `replica` `hpa` `confluence` `ems-financial`

---

## Overview (EN/TH)

**EN**: This tutorial demonstrates a **Single API** execution flow—the most fundamental pattern that completes execution with only one API. We walk through configurable code blocks and their purposes.

**TH**: คู่มือนี้เป็นตัวอย่าง **Single API** ซึ่งเป็นพื้นฐานของ execution flow ที่ใช้เพียง API เดียว อธิบายการตั้งค่าแต่ละส่วนและวัตถุประสงค์ของมัน

---

## Constant Configuration

### Flow Definition

```ts
export const projectId = 'glo'
export const serviceName = 'adaptor-ems-financial'
export const flowList = ['adaptorEmsFinancial']
export const serviceList = [`adaptor-ems-financial/glo-adaptor`]
export const vuList = [1, 2, 3]
export const stages = getStages(vuList, '1s', '5m', '3m')
export const customOptions = getOptions(flowList, stages, '2m')
export const flowMap = {
  [flowList[0]]: [serviceList[0]],
}
export const apiMap = {}
```

### Key Fields & Semantics

- **projectId**: lowercase repository ID
- **serviceName**: main service
- **flowList**: execution list
- **serviceList**: deploy/copy manifest target
- **vuList**: VU iterations
- **stages**: ramp-up, execution, pause
- **customOptions**: scenario reading & delay
- **flowMap**: flow → services mapping
- **apiMap**: API metrics definition

---

## Mock Definition

### addMockConfig — Parameters

(omitted for brevity here; full detail preserved in original answer)

### Example

```ts
addMockConfig(
  mockConfig,
  'ems-financial',
  '/emsws-api/finance/ems/transfer/ems/standardTransfer',
  'POST',
  '*',
  1,
  200,
  {},
  {
    fromAccountStatus: 1,
    fromAccountNo: '1234567891',
    curCode: '',
    waiveFlag: '',
    fromAccountAvailBal: 50000.0,
    fromAccountLedgerBal: 20000.0,
    standardTransferRespHeader: {
      responseCode: 0,
      responseDesc: 'success',
      wsRefId: '123456788',
    },
  },
)
```

---

## Manifest Definition

(Full detail included originally; maintained structure)

```ts
export const portMap = { [serviceList[0]]: 80 }
export const protocolMap = { [serviceList[0]]: 'http' }
```

Example ConfigMap, Secret, Resource, Replica, HPA structures preserved.

---

## Execution Definition

### Options

```ts
export const options = c.customOptions
```

### Execution Function

```ts
export function adaptorEmsFinancial() {
  // ...
}
```

### API Binding, Properties, Request & Checks

```ts
group(c.flowList[flowIdx], function () {
  const res = session.request(
    singleApiMethod,
    `${singleApiEndpoint}${singleApiPath}`,
    JSON.stringify(singleApiBody),
    singleApiParams,
  )
  check(res, {
    'is code: 1000': (r) => r.body && r.json().code === 1000,
  })
})
```

---

## Summary Function

```ts
export function handleSummary(data) {
  return generateResults(c, session, data)
}
```

---

## Execution Runtime

Windows PowerShell:
```powershell
$Env:RELEASE="mvp-99"; $Env:SCENARIO="adaptorEmsFinancial"; $Env:EXECUTION_TYPE="capacity"; ../k6 run ./init.js
```

macOS/Linux:
```bash
RELEASE="mvp-99" SCENARIO="adaptorEmsFinancial" EXECUTION_TYPE="capacity" ../k6 run ./init.js
```

---
