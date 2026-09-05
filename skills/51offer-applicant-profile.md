---
name: 51offer-applicant-profile
description: Build and maintain a 51offer applicant profile — basic details, education history, language scores, work experience and application materials — with the delete operation that reverses each write.
api: 51offer Horizon Site API
base_url: https://www.51offer.com
operations:
  - getUserBasicInfo
  - saveUserBasicInfo
  - getUserEducationInfo
  - saveUserEducationInfo
  - deleteUserEducationInfo
  - getUserLanguageScoreInfo
  - saveUserLanguageScoreInfo
  - deleteUserLanguageScoreInfo
  - getUserWorkInfo
  - saveUserWorkInfo
  - deleteUserWorkInfo
  - getUserPSInfo
  - saveUserPSInfo
  - deleteUserPSInfo
  - deleteMaterialFileInfo
generated: '2026-09-05'
method: generated
source: openapi/51offer-horizon-site-openapi.yml (converted from https://www.51offer.com/api-docs)
---

# Applicant profile and materials

The `diym` controllers hold everything a UK/US/AU application is assembled from. The shape is
uniform: a `query` read, a `save` write, and — for the repeatable sections — a `delete` that
reverses it.

## Personal profile

| Section | Read | Write | Reverse |
|---|---|---|---|
| Basic details | `GET /diym/basic/query/basicinfo` (`getUserBasicInfo`) | `POST /diym/basic/save/basicinfo` (`saveUserBasicInfo`) | — |
| Education | `GET /diym/basic/query/education` (`getUserEducationInfo`) | `POST /diym/basic/save/education` (`saveUserEducationInfo`) | `GET /diym/basic/delete/education` (`deleteUserEducationInfo`) |
| Language scores | `GET /diym/basic/query/language` (`getUserLanguageScoreInfo`) | `POST /diym/basic/save/language` (`saveUserLanguageScoreInfo`) | `GET /diym/basic/delete/language` (`deleteUserLanguageScoreInfo`) |
| Work experience | `GET /diym/basic/query/work` (`getUserWorkInfo`) | `POST /diym/basic/save/work` (`saveUserWorkInfo`) | `GET /diym/basic/delete/work` (`deleteUserWorkInfo`) |

The three repeatable sections take a JSON **array** body (`UsersEducationVO[]`,
`UsersLanguageScoreVO[]`, `UsersWorkVO[]`), so a save replaces the whole list. Read first, merge,
then write — otherwise a save silently drops entries you did not send.

Address reference data comes from `getProvinceByCountry`, `getCityByProvince` and
`getCountyByCity` under `/diym/apply/`.

## Per-school materials

    GET  /diym/school/query/ps               getUserPSInfo             # personal statement
    POST /diym/school/save/ps                saveUserPSInfo
    GET  /diym/school/delete/ps              deleteUserPSInfo
    GET  /diym/school/query/reference        getUserSuborderReferenceInfo
    POST /diym/school/save/reference         saveUserSuborderReferenceInfo
    GET  /diym/school/delete/reference       deleteUserSuborderReferenceInfo
    GET  /diym/school/query/suborderother    getUserSuborderOtherInfo
    POST /diym/school/save/suborderother     saveUserSuborderOtherInfo
    GET  /diym/school/delete/suborderother   deleteUserSuborderOtherInfo
    GET  /diym/apply/delete/file             deleteMaterialFileInfo    # remove an uploaded file

## Application state

    POST /diym/apply/query/apply         getUserMainApplyInfo
    POST /diym/apply/query/mainStatus    getUserMainApplyStatus
    GET  /diym/apply/query/list          getApplySchoolList
    GET  /diym/apply/query/status        getOrderSubmitStatus
    POST /diym/apply/save/status         updateOrderSubmitStatus

## Rules that apply to every call

- **Reversibility is `documented`, not `verified`.** Every delete above exists in the contract, but
  51offer publishes **no window** — nothing states how long after a save or a submission a delete
  still applies. Do not tell a user a deadline; none is published. See the `reversibility` block in
  `conventions/51offer-conventions.yml`.
- **Delete operations are declared `GET`.** Never issue them speculatively or from a prefetch.
- **No idempotency.** A repeated `saveUser*Info` replaces the list; a repeated
  `updateOrderSubmitStatus` re-applies the state change with no request key to deduplicate on.
- **Authentication** is the `token` header, undocumented by the provider
  (`authentication/51offer-authentication.yml`).
- Branch on the envelope `code`, not the HTTP status.
