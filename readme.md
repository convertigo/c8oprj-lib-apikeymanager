


# lib_ApiKeyManager

# API Key Manager

Centralized API key lifecycle management for applications that expose or protect APIs.

## Purpose

This library creates and manages API keys that an application can use to control access to its API endpoints. It is designed to support applications based on **Convertigo URL Mapper**, where an incoming request can be checked before access to a mapped resource is granted.

## Key lifecycle

- **Generate a key** with a client identifier, an optional label, one or more scopes, and an optional expiration date.
- **Store securely**: only the key's SHA-256 fingerprint and metadata are stored in FullSync. The raw key is returned only when it is generated and is never persisted.
- **Validate a key** by checking its fingerprint, active status, expiration date, and an optional required scope.
- **List keys** to retrieve administrative metadata without exposing raw key values.
- **Revoke a key** to immediately mark it inactive.
- **Reactivate a key** when access must be restored.

## URL Mapper integration

A URL Mapper based application can call `validate_api_key` while processing a protected route. The application provides the API key received from the client and, when appropriate, the scope required by the route. Validation succeeds only when a matching key is active, unexpired, and authorized for that scope.

This keeps authorization rules reusable and separate from individual URL mappings.

## Example

The **UrlMapperDemo** project is an example consumer. It can use this library to protect mapped resources with API keys and to enforce scopes such as `resources:read`.

## Security

Administrative sequences (`generate_api_key`, `list_api_keys`, `revoke_api_key`, and `reactivate_api_key`) require an authenticated context.

`validate_api_key` is intended for request-time validation and **never persists the raw API key**.

## Scopes

Provide scopes as a space-separated list, for example: `resources:read`.




For more technical informations : [documentation](./project.md)

- [Installation](#installation)
- [Mobile Library](#mobile-library)


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



