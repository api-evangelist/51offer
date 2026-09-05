---
name: 51offer-diy-school-selection
description: Walk a student through 51offer's DIY school-selection flow — background capture, intelligent matching, the intent cart, and submitting an intent list for review.
api: 51offer Horizon Site API
base_url: https://www.51offer.com
operations:
  - getOneStep
  - saveUsersOneSteps
  - saveUsersThreesteps
  - getUsersThreestepsIsFull
  - queryApplyCountByUniversity
  - getSchoolList
  - addCart
  - getCartCountry
  - deleteCartSchool
  - deleteCartMajor
  - confirmIntentions
  - validateGoods
  - submitIntentions
generated: '2026-09-05'
method: generated
source: openapi/51offer-horizon-site-openapi.yml (converted from https://www.51offer.com/api-docs)
---

# DIY school selection

The `ngdiyselectschool` controller is 51offer's self-service selection funnel: capture the
student's background, get matched schools, build an intent cart, submit it for review. Every
operationId below is copied from the contract.

## 1. Capture the background ("滑" steps)

    GET  /ngdiyselectschool/getOneStep              # getOneStep — first-slide data
    GET  /ngdiyselectschool/saveUsersOneSteps       # saveUsersOneSteps — save slides 1 and 2
    POST /ngdiyselectschool/saveUsersThreesteps     # saveUsersThreesteps — save slide 3
    GET  /ngdiyselectschool/getUsersThreestepsIsFull  # getUsersThreestepsIsFull — is the profile complete?
    GET  /ngdiyselectschool/getUsersThreestepsVO     # getUsersThreestepsVO — read it back

Reference data for the form comes from `getCoursesType`, `getLanguageType`, `getMajorDirection`
(broad category), `getMajorCategoryByDirectionId` (sub-category), `getCurrentSchoolsByName`
(fuzzy search of the student's current school) and `readSearchChinaSchoolMajorXML2String`.

Gate the next step on `getUsersThreestepsIsFull` rather than assuming the save worked.

## 2. Match

    GET /ngdiyselectschool/intelligent    # queryApplyCountByUniversity — intelligent matching from the saved background
    GET /ngdiyselectschool/getSchoolList  # getSchoolList — plain query, smart sort, or keyword search
    GET /ngdiyselectschool/getJoinMajor   # getJoinMajor — pathway/articulation majors

## 3. Build the intent cart

    GET /ngdiyselectschool/addCart              # addCart
    GET /ngdiyselectschool/getCartCountry       # getCartCountry — hides already-submitted intents
    GET /ngdiyselectschool/getCartCountry2      # getCartCountry2 — shows all intents
    GET /ngdiyselectschool/getCartCountryCount  # getCartCountryCount

**Reversal:** `deleteCartSchool` and `deleteCartMajor` remove an intent. `deleteCartMajor` accepts
two addressing modes, and the provider says so in the operation summary: by `id`, or by the
`degreeId` + `schoolId` + `majorId` triple.

## 4. Confirm and submit

    GET /ngdiyselectschool/validateGoods       # validateGoods — check the cart can be confirmed
    GET /ngdiyselectschool/confirmIntentions   # confirmIntentions
    GET /ngdiyselectschool/submitIntentions    # submitIntentions — send for review

Then the `ngdiychooseSchool` controller takes over: `getChooseSchoolList` (awaiting confirmation),
`enterRecommend` (confirm the application), `generateSnapshot`, `viewFinalMaterial` and
`finalMaterial` (final submission).

## Rules that apply to every call

- **Authentication.** These are per-user operations. The only auth signal 51offer exposes is a
  `token` request header (`Access-Control-Allow-Headers: token`); there is no published auth
  documentation, no OAuth and no scopes. See `authentication/51offer-authentication.yml`.
- **Mutating operations use GET.** `addCart`, `deleteCartSchool`, `confirmIntentions` and
  `submitIntentions` are all declared `GET` in the provider's own contract. Treat them as writes
  regardless of the verb: never prefetch them, never retry them blind.
- **No idempotency, anywhere.** There is no request key on any of the 118 mutating operations. A
  retried `submitIntentions` is a second submission.
- **`validateGoods` before `confirmIntentions`** is the closest thing this API has to a dry run;
  there is no preview or simulation mode.
- Branch on the envelope's `code` field, not the HTTP status — every response is HTTP 200.
