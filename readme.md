


# lib_ApiKeyManager

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



For more technical informations : [documentation](./project.md)

- [Installation](#installation)
- [Mobile Library](#mobile-library)
    - [Shared Components](#shared-components)
        - [ApiKeyAccessGuard](#apikeyaccessguard)
        - [ApiKeyCreateForm](#apikeycreateform)
        - [ApiKeyList](#apikeylist)
        - [ApiKeyManagerPanel](#apikeymanagerpanel)


## Installation

1. In your Convertigo Studio click on ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/icons/studio/project_import.gif?raw=true "Import a project in treeview") to import a project in the treeview
2. In the import wizard

   ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/tomcat/webapps/convertigo/templates/ftl/project_import_wzd.png?raw=true "Import Project")
   
   paste the text below into the `Project remote URL` field:
   <table>
     <tr><td>Usage</td><td>Click the copy button at the end of the line</td></tr>
     <tr><td>To contribute</td><td>

     ```
     lib_ApiKeyManager=https://github.com/convertigo/c8oprj-lib-apikeymanager.git:branch=master
     ```
     </td></tr>
     <tr><td>To simply use</td><td>

     ```
     lib_ApiKeyManager=https://github.com/convertigo/c8oprj-lib-apikeymanager/archive/master.zip
     ```
     </td></tr>
    </table>
3. Click the `Finish` button. This will automatically import the __lib_ApiKeyManager__ project


## Mobile Library

Describes the mobile application global properties

### Shared Components

#### ApiKeyAccessGuard

Checks the API Key Manager administrator authorization (Convertigo WEB_ADMIN session or role delegated by the host application) and shows an explanatory message when the session is not authorized. Input: application. Event: authorizationChecked {authorized, authority, message}.

**variables**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>application</td><td>Host application identifier used to scope API keys.</td>
</tr>
</table>

**events**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>authorizationChecked</td><td>Emitted once the authorization is known, with {authorized, authority, message}.</td>
</tr>
</table>

#### ApiKeyCreateForm

Autonomous API key creation form with one-time display of the generated secret. Inputs: application, defaultClientId, defaultLabel (initial form values). Event: keyCreated {application, result}.

**variables**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>application</td><td>Host application identifier the new key belongs to.</td>
</tr>
<tr>
<td>defaultClientId</td><td>Initial client identifier value.</td>
</tr>
<tr>
<td>defaultLabel</td><td>Initial label value.</td>
</tr>
</table>

**events**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>keyCreated</td><td>Emitted after a key is generated, with {application, result}.</td>
</tr>
</table>

#### ApiKeyList

Autonomous API key list for one host application: filter, refresh, revoke and reactivate. Inputs: application, filter, refreshToken (change it, e.g. Date.now(), to force a reload). Events: keysLoaded {application,count}, keyRevoked {id,application}, keyReactivated {id,application}.

**variables**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>application</td><td>Host application identifier. Keys are scoped by it.</td>
</tr>
<tr>
<td>filter</td><td>Initial status filter: active, revoked, expired or all.</td>
</tr>
<tr>
<td>refreshToken</td><td>Change this value (for example Date.now()) to force the list to reload.</td>
</tr>
</table>

**events**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>keyReactivated</td><td>Emitted after a key is reactivated, with {id, application}.</td>
</tr>
<tr>
<td>keyRevoked</td><td>Emitted after a key is revoked, with {id, application}.</td>
</tr>
<tr>
<td>keysLoaded</td><td>Emitted after the key list is loaded, with {application, count}.</td>
</tr>
</table>

#### ApiKeyManagerPanel

Drop-in API key management panel: creation form plus key list, wired together. Input: application. Events: keyCreated, keyRevoked, keyReactivated.

**variables**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>application</td><td>Host application identifier.</td>
</tr>
<tr>
<td>defaultClientId</td><td>Initial client identifier shown in the creation form.</td>
</tr>
<tr>
<td>defaultLabel</td><td>Initial label shown in the creation form.</td>
</tr>
</table>

**events**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>keyCreated</td><td></td>
</tr>
<tr>
<td>keyReactivated</td><td></td>
</tr>
<tr>
<td>keyRevoked</td><td></td>
</tr>
</table>



