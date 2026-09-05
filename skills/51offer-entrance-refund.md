---
name: 51offer-entrance-refund
description: File and track a 51offer entrance refund — the only money-reversal path the API exposes — and read the order it reverses.
api: 51offer Horizon Site API
base_url: https://www.51offer.com
operations:
  - getEntranceRefund
  - submitEntranceRefund
  - getRefundProgress
  - userOrder
  - userOrderDetails
  - orderPayOperationRecord
generated: '2026-09-05'
method: generated
source: openapi/51offer-horizon-site-openapi.yml (converted from https://www.51offer.com/api-docs)
---

# Entrance refund

`/ngrefund` is the only reversal path for money in this API. It is an **application**, not a void:
you submit a request and then poll its progress through 51offer's review.

## 1. Read the order being reversed

    GET /ngMall/userOrder                  userOrder                  # the user's orders
    GET /ngMall/userOrderDetails           userOrderDetails
    GET /ngMall/orderPayOperationRecord    orderPayOperationRecord    # payment operation history for an order code

## 2. Read the existing refund application before filing another

    GET /ngrefund/info      getEntranceRefund     # 获取入学退款申请

Call this first. There is no idempotency key on `submitEntranceRefund`, so this read is the only
duplicate protection available — if an application already exists, do not file a second one.

## 3. File

    POST /ngrefund/sub      submitEntranceRefund  # 提交入学退款申请

## 4. Track

    GET /ngrefund/progress  getRefundProgress     # 获取退款进度

## What is NOT published — do not fill it in

- **No refund window.** Nothing 51offer publishes states a deadline, an eligibility period, or a
  cut-off after enrolment. Never quote one to a user; an invented window here costs real money.
- **No refund amount rules, fees or proration** appear anywhere in the contract or on the site.
- **No decline or rejection code reference.** If the application is refused, the envelope's
  `message` (Chinese prose) is all there is.
- **No webhook or callback** for refund state. `getRefundProgress` polling is the only mechanism;
  the one callback in this API, `signSuccessNotice` on `/ngMall`, is for contract signing, not
  refunds.

## Rules that apply to every call

- **No idempotency on the write.** `POST /ngrefund/sub` has no request key. A retry after a timeout
  may file a second application — read `/ngrefund/info` before retrying.
- **No rate-limit signal** and **no error-code reference**; branch on the envelope `code`.
- **Authentication** is the undocumented `token` header.
