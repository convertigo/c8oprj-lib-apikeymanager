


# lib_ApiKeyManager

Centralized API key management library.

## Capabilities

- Generates API keys and stores only their **SHA-256 fingerprint** and metadata in FullSync.
- Validates active, unexpired keys with optional scope enforcement.
- Lists key metadata and supports key revocation and reactivation.

## Security

Administrative sequences—generation, listing, revocation, and reactivation—require an authenticated context.

`validate_api_key` is intended for validation calls and **never persists the raw API key**.

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



