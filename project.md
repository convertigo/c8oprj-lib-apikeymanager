
# ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/project_color_16x16.png?raw=true "Project") lib_ApiKeyManager

# API Key Manager

Centralized API key lifecycle management for applications that expose or protect APIs.

## Purpose

This library creates and manages API keys that an application can use to control access to its API endpoints. It is designed to support applications based on **Convertigo URL Mapper**, where an incoming request can be checked before access to a mapped resource is granted.

## Key lifecycle

- **Generate a key** with a client identifier, an optional label, one or more scopes, an optional expiration date and the owning `application`.
- **Store securely**: only the key's SHA-256 fingerprint and metadata are stored in FullSync. The raw key is returned only when it is generated and is never persisted.
- **Validate a key** by checking its fingerprint, active status, expiration date, an optional required scope and an optional `application`.
- **List keys** of one application to retrieve administrative metadata without exposing raw key values.
- **Revoke a key** to immediately mark it inactive.
- **Reactivate a key** when access must be restored.

## Multi-application isolation

Every key carries an `application` field (default `default`). `generate_api_key`, `list_api_keys` and `validate_api_key` accept an `application` variable so that several host applications can share this library and the same FullSync store without seeing each other's keys. Legacy keys without the field are treated as `default`. An empty or missing `application` on `list_api_keys` and `validate_api_key` means no application filter.

## Security and authorization

There is no login screen in this library. Authorization is read from the HTTP session, with the same method as lib_ConvertigoMCP. Administrative sequences (`generate_api_key`, `list_api_keys`, `revoke_api_key`, `reactivate_api_key`) require an authenticated context **and** API Key Manager administrator privileges, granted by either:

- a Convertigo `WEB_ADMIN` session (application opened from the administration console), or
- a role delegated by the host application: call `lib_ApiKeyManager.delegate_admin_role` (Private, from a Call Sequence step) in your own login sequence after you authenticated the user and verified it may administer API keys; call `revoke_admin_role` from your logout sequence. The role is stored in the session attribute `apiKeyManager.role`.

`admin_status` (Hidden, no authentication required) reports the current authorization: `authorized`, `authority` (`web_admin` or `delegated`) and a user-facing `message`.

The FullSync transactions are Private, and all of them except `find_api_key_by_hash` require an authenticated context; clients never call them directly. `validate_api_key` is intended for request-time validation, is callable without a session, and **never persists the raw API key**.

## Shared components for host applications

Reference this project and use the exposed NGX shared components. Component inputs are read with `this.<input>` inside the component; emitted event data reaches the host handler in `out`.

- **ApiKeyAccessGuard** (input `application`; event `authorizationChecked` with `{authorized, authority, message}`): calls `admin_status` and shows the explanatory message when the session is not authorized. Store `out.authorized` in a page local and hide your own content until it is true.
- **ApiKeyList** (inputs `application`, `filter`, `refreshToken`; events `keysLoaded {application, count}`, `keyRevoked {id, application}`, `keyReactivated {id, application}`): autonomous list with status filter, refresh, revoke and reactivate. Loads itself on init; change `refreshToken` (for example `Date.now()`) to force a reload.
- **ApiKeyCreateForm** (inputs `application`, `defaultClientId`, `defaultLabel`; event `keyCreated {application, result}`): creation form with one-time display of the generated secret. `defaultClientId` and `defaultLabel` prefill the form; expiration defaults to one year and all scopes are checked.
- **ApiKeyManagerPanel** (inputs `application`, `defaultClientId`, `defaultLabel`; events `keyCreated`, `keyRevoked`, `keyReactivated`): drop-in panel assembling the form and the list, and reloading the list after each creation.

The library's own `Page` is the reference integration: `ApiKeyAccessGuard`, then `ApiKeyManagerPanel` shown when `authorized` is true.

## Running the library's own page

- Development: open the live viewer (`http://localhost:46010/home` in Studio) in the same browser as the administration console; the Convertigo session cookie is shared on `localhost` across ports, so the `WEB_ADMIN` session applies.
- Production: the administration console opens `DisplayObjects/mobile/home`, which is the **production build**. It is only refreshed by the Studio production build of the NGX application; rebuild it after changing the UI, otherwise an outdated page is served.

## URL Mapper integration

A URL Mapper based application can call `validate_api_key` while processing a protected route. The application provides the API key received from the client, the scope required by the route and, when appropriate, its `application` name. Validation succeeds only when a matching key is active, unexpired, authorized for that scope and owned by that application.

## Example

The **UrlMapperDemo** project is an example consumer. It can use this library to protect mapped resources with API keys and to enforce scopes such as `resources:read`.

## Scopes

Provide scopes as a space-separated list, for example: `resources:read`.


<details><summary><span style="color:DarkGoldenRod"><i>Connectors</i></span></summary><blockquote><p>


<details><summary><b>api_keys_store</b> : Internal API key repository compatible with UrlMapperDemo: stores SHA-256 fingerprints, scopes, status, expiration, and audit metadata</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/connectors/images/fullsyncconnector_color_16x16.png?raw=true "FullSyncConnector") api_keys_store

Internal API key repository compatible with UrlMapperDemo: stores SHA-256 fingerprints, scopes, status, expiration, and audit metadata. Never store raw API keys here.

<details><summary><span style="color:DarkGoldenRod"><i>Transactions</i></span></summary><blockquote><p>


<details><summary><b>create_api_key</b> : Internal API key provisioning: stores keyHash only, never the raw key</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postdocument_color_16x16.png?raw=true "PostDocumentTransaction") create_api_key

Internal API key provisioning: stores keyHash only, never the raw key.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;active
</td>
<td>
API key active/inactive status.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Host application identifier owning this key (multi-application isolation).
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;clientId
</td>
<td>
Owning client identifier.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;createdAt
</td>
<td>
ISO-8601 creation timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;expiresAt
</td>
<td>
ISO-8601 expiration timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;keyHash
</td>
<td>
SHA-256 fingerprint of the raw API key. The raw key is never stored.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;label
</td>
<td>
Human-readable label displayed in the manager UI.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;revokedAt
</td>
<td>
ISO-8601 revocation timestamp, empty while active.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/multivaluedvariable_color_16x16.png?raw=true "  alt="RequestableMultiValuedVariable" >&nbsp;scopeList
</td>
<td>
Scopes as an array for validation convenience; scopes string remains the compatibility field.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;scopes
</td>
<td>
Space-separated authorization scopes compatible with UrlMapperDemo.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;type
</td>
<td>
Document type marker.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>find_api_key_by_hash</b> : Internal lookup of an active API key by SHA-256 fingerprint</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postfind_color_16x16.png?raw=true "PostFindTransaction") find_api_key_by_hash

Internal lookup of an active API key by SHA-256 fingerprint. Raw keys must never be sent to FullSync.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Optional host application the key must belong to. 'default' also matches legacy keys without application.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;keyHash
</td>
<td>
SHA-256 fingerprint of the supplied API key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;now
</td>
<td>
Current ISO-8601 timestamp for expiration filtering.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;requiredScope
</td>
<td>
Optional required scope for validation.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>initialize_api_keys_database</b> : Initializes the dedicated API key database</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/resetdatabase_color_16x16.png?raw=true "ResetDatabaseTransaction") initialize_api_keys_database

Initializes the dedicated API key database. Do not run on production data.
</p></blockquote></details>

<details><summary><b>list_api_keys</b> : Lists API key metadata without exposing raw API keys</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postfind_color_16x16.png?raw=true "PostFindTransaction") list_api_keys

Lists API key metadata without exposing raw API keys.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Optional host application filter. 'default' also matches legacy keys without application.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>update_api_key</b> : Updates API key metadata by document id; used for revoke, reactivate and scope/expiration changes</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postdocument_color_16x16.png?raw=true "PostDocumentTransaction") update_api_key

Updates API key metadata by document id; used for revoke, reactivate and scope/expiration changes.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;_id
</td>
<td>
FullSync document id to update.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;active
</td>
<td>
API key active/inactive status.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;expiresAt
</td>
<td>
Updated ISO-8601 expiration timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;revokedAt
</td>
<td>
ISO-8601 revocation timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;scopes
</td>
<td>
Updated space-separated scopes.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;updatedAt
</td>
<td>
ISO-8601 update timestamp.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><b>void</b> : Placeholder SQL connector for template projects</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/connectors/images/sqlconnector_color_16x16.png?raw=true "SqlConnector") void

Placeholder SQL connector for template projects

<details><summary><span style="color:DarkGoldenRod"><i>Transactions</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") void

Placeholder transaction that intentionally cancels execution
</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Sequences</i></span></summary><blockquote><p>


<details><summary><b>admin_status</b> : Reports whether the current HTTP session holds the Convertigo WEB_ADMIN role (same method as lib_ConvertigoMCP</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") admin_status

Reports whether the current HTTP session holds the Convertigo WEB_ADMIN role (same method as lib_ConvertigoMCP.McpAdminStatus).
</p></blockquote></details>

<details><summary><b>delegate_admin_role</b> : Host application integration: call this sequence (Call Sequence step) from your own login sequence, after you authenticated the user and verified it may administer API keys</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") delegate_admin_role

Host application integration: call this sequence (Call Sequence step) from your own login sequence, after you authenticated the user and verified it may administer API keys. It marks the current session as API Key Manager administrator (session attribute apiKeyManager.role=admin).
</p></blockquote></details>

<details><summary><b>generate_api_key</b> : Generates a raw API key once, stores only its SHA-256 fingerprint, and returns metadata plus the raw secret one time</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") generate_api_key

Generates a raw API key once, stores only its SHA-256 fingerprint, and returns metadata plus the raw secret one time.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Host application identifier owning this key. Defaults to 'default'.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;clientId
</td>
<td>
Stable client identifier.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;expiresAt
</td>
<td>
ISO-8601 expiration timestamp. Defaults to one year from now when empty.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;label
</td>
<td>
Human-readable key label.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;scopes
</td>
<td>
Space-separated scopes compatible with UrlMapperDemo.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>list_api_keys</b> : Lists API key metadata without exposing raw API keys</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") list_api_keys

Lists API key metadata without exposing raw API keys.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Optional host application filter. Empty lists every key; 'default' also includes legacy keys.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>reactivate_api_key</b> : Reactivates an existing API key document</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") reactivate_api_key

Reactivates an existing API key document.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;_id
</td>
<td>
FullSync API key document id.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>revoke_admin_role</b> : Host application integration: call this sequence from your logout sequence to remove the delegated API Key Manager administrator role from the session</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") revoke_admin_role

Host application integration: call this sequence from your logout sequence to remove the delegated API Key Manager administrator role from the session.
</p></blockquote></details>

<details><summary><b>revoke_api_key</b> : Revokes an API key by marking the FullSync document inactive</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") revoke_api_key

Revokes an API key by marking the FullSync document inactive.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;_id
</td>
<td>
FullSync API key document id.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>validate_api_key</b> : Validates a raw API key against the FullSync SHA-256 fingerprint store and an optional required scope</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") validate_api_key

Validates a raw API key against the FullSync SHA-256 fingerprint store and an optional required scope.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
Raw API key to validate. It is hashed before lookup and never stored.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;application
</td>
<td>
Optional host application the key must belong to.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;requiredScope
</td>
<td>
Optional scope required by the caller.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Mobile Application</i></span></summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/mobileapplication_color_16x16.png?raw=true "MobileApplication") Application

Describes the mobile application global properties

<details><summary><span style="color:DarkGoldenRod"><i>Pages</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/pagecomponent_color_16x16.png?raw=true "PageComponent") Page

API key manager dashboard
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Shared Components</i></span></summary><blockquote><p>


<details><summary><b>ApiKeyAccessGuard</b> : Checks the API Key Manager administrator authorization (Convertigo WEB_ADMIN session or role delegated by the host application) and shows an explanatory message when the session is not authorized</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uisharedcomponent_16x16.png?raw=true "UISharedRegularComponent") ApiKeyAccessGuard

Checks the API Key Manager administrator authorization (Convertigo WEB_ADMIN session or role delegated by the host application) and shows an explanatory message when the session is not authorized. Input: application. Event: authorizationChecked {authorized, authority, message}.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;application
</td>
<td>
Host application identifier used to scope API keys.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Events</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;authorizationChecked
</td>
<td>
Emitted once the authorization is known, with {authorized, authority, message}.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>ApiKeyCreateForm</b> : Autonomous API key creation form with one-time display of the generated secret</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uisharedcomponent_16x16.png?raw=true "UISharedRegularComponent") ApiKeyCreateForm

Autonomous API key creation form with one-time display of the generated secret. Inputs: application, defaultClientId, defaultLabel (initial form values). Event: keyCreated {application, result}.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;application
</td>
<td>
Host application identifier the new key belongs to.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;defaultClientId
</td>
<td>
Initial client identifier value.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;defaultLabel
</td>
<td>
Initial label value.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Events</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyCreated
</td>
<td>
Emitted after a key is generated, with {application, result}.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>ApiKeyList</b> : Autonomous API key list for one host application: filter, refresh, revoke and reactivate</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uisharedcomponent_16x16.png?raw=true "UISharedRegularComponent") ApiKeyList

Autonomous API key list for one host application: filter, refresh, revoke and reactivate. Inputs: application, filter, refreshToken (change it, e.g. Date.now(), to force a reload). Events: keysLoaded {application,count}, keyRevoked {id,application}, keyReactivated {id,application}.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;application
</td>
<td>
Host application identifier. Keys are scoped by it.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;filter
</td>
<td>
Initial status filter: active, revoked, expired or all.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;refreshToken
</td>
<td>
Change this value (for example Date.now()) to force the list to reload.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Events</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyReactivated
</td>
<td>
Emitted after a key is reactivated, with {id, application}.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyRevoked
</td>
<td>
Emitted after a key is revoked, with {id, application}.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keysLoaded
</td>
<td>
Emitted after the key list is loaded, with {application, count}.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>ApiKeyManagerPanel</b> : Drop-in API key management panel: creation form plus key list, wired together</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uisharedcomponent_16x16.png?raw=true "UISharedRegularComponent") ApiKeyManagerPanel

Drop-in API key management panel: creation form plus key list, wired together. Input: application. Events: keyCreated, keyRevoked, keyReactivated.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;application
</td>
<td>
Host application identifier.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;defaultClientId
</td>
<td>
Initial client identifier shown in the creation form.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;defaultLabel
</td>
<td>
Initial label shown in the creation form.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Events</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyCreated
</td>
<td>

</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyReactivated
</td>
<td>

</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;keyRevoked
</td>
<td>

</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>
