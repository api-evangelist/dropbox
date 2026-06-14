# Dropbox GraphQL Schema

## Overview

This conceptual GraphQL schema models the Dropbox cloud file storage REST API
(https://www.dropbox.com/developers/documentation/http/documentation). It covers
files, folders, metadata, sharing, teams, Paper documents, search, and
authentication concepts exposed by the Dropbox API v2 at
`https://api.dropboxapi.com/2/`.

The schema is derived from the official Dropbox API HTTP reference and the
open-source API specification at https://github.com/dropbox/dropbox-api-spec.

---

## Source

- REST API base URL: `https://api.dropboxapi.com/2/`
- Documentation: https://www.dropbox.com/developers/documentation/http/documentation
- API spec repo: https://github.com/dropbox/dropbox-api-spec
- GitHub org: https://github.com/dropbox

---

## Type Index

### File and Folder Metadata
- `File` — top-level union of file, folder, and deleted entries
- `FileMetadata` — metadata for a file stored in Dropbox
- `FolderMetadata` — metadata for a folder stored in Dropbox
- `DeletedMetadata` — metadata for a deleted file or folder
- `Dimensions` — pixel width and height for image/video media
- `GpsCoordinates` — latitude and longitude extracted from EXIF data
- `MediaInfo` — union of photo and video metadata
- `PhotoMetadata` — EXIF metadata for photo files
- `VideoMetadata` — duration and dimensions for video files

### Upload / Commit
- `CommitInfo` — parameters for uploading a file (path, mode, autorename)
- `WriteMode` — enum: add, overwrite, update
- `UploadSessionCursor` — session ID and byte offset for chunked uploads
- `UploadSessionFinishArg` — commit info tied to an upload session

### File Properties
- `PropertyGroup` — named set of custom key-value fields on a file
- `PropertyField` — individual key-value pair within a property group
- `PropertyGroupTemplate` — schema definition for a set of property fields
- `PropertyFieldTemplate` — definition of a single property field in a template

### Tags
- `Tag` — user-defined string tag attached to a file or folder

### Search
- `SearchResult` — paginated list of search result matches
- `SearchResultMatch` — individual match from a search query
- `SearchResultMetadata` — union of file, folder, or deleted metadata for a match
- `SearchMode` — enum: filename, filename_and_content, deleted_filename
- `SearchMatchType` — enum: filename, content, both
- `SearchOptions` — configurable parameters for a search request

### Sharing — Shared Folders
- `SharedFolder` — a Dropbox folder shared with other users or groups
- `SharedFolderPolicy` — access and membership policies on a shared folder
- `MemberPolicy` — enum: team, anyone
- `AclUpdatePolicy` — enum: owner, editors
- `ViewerInfoPolicy` — enum: enabled, disabled
- `SyncSetting` — enum: default, not_synced, not_synced_inactive

### Sharing — Shared Files
- `SharedFile` — a single file shared via a Dropbox shared link
- `SharedFilePolicy` — policy controlling who can comment and access

### Shared Links
- `LinkMetadata` — base metadata for a shared link
- `FileLinkMetadata` — shared link pointing to a single file
- `FolderLinkMetadata` — shared link pointing to a folder
- `SharedLinkSettings` — settings applied when creating a shared link
- `LinkPermissions` — viewer permissions on a shared link
- `SharedLinkPolicy` — enum: anyone, team, no_one

### Permissions and Access
- `Permission` — an action-level permission on a shared folder or file
- `Visibility` — enum: public, team_only, password, team_and_password, shared_folder_only
- `AccessLevel` — enum: owner, editor, viewer, viewer_no_comment
- `MemberAccess` — pairing of a member with their access level

### Paper
- `PaperDoc` — a Dropbox Paper collaborative document
- `PaperFolder` — a folder grouping Paper documents
- `PaperDocPermissionLevel` — enum: edit, view_and_comment, view_only

### Users
- `User` — a Dropbox account (individual or managed)
- `FullAccount` — complete account info for the authenticated user
- `BasicAccount` — minimal account info for any Dropbox user
- `SpaceUsage` — storage allocation and usage for a user account
- `SpaceAllocation` — union of individual and team space allocation

### Team
- `TeamMemberProfile` — full profile of a member in a Dropbox team
- `TeamMember` — member record including role and status
- `TeamMemberStatus` — enum: active, invited, suspended, removed
- `Member` — union of team member and external (non-team) user
- `TeamFolderMetadata` — metadata for a team-owned folder
- `GroupInfo` — summary information about a Dropbox group
- `GroupMembership` — record of a user belonging to a group
- `GroupAccessType` — enum: member, owner
- `TeamSpace` — the root namespace of a team
- `Namespace` — a logical namespace (team folder, app folder, or shared folder)
- `TeamReport` — usage and activity report for a team

### Admin
- `Admin` — a team member with elevated administrative privileges
- `AdminTier` — enum: team_admin, user_management_admin, support_admin, member_only

### Authentication
- `AuthToken` — an active Dropbox API access token
- `OAuthToken` — OAuth 2.0 token including access token, token type, and expiry
- `TokenScopeError` — error returned when a required OAuth scope is missing

### Legal Holds
- `LegalHold` — a legal hold applied to team member content
- `LegalHoldPolicy` — policy definition for a legal hold

### Signatures (Dropbox Sign)
- `Signature` — a completed eSignature on a document
- `SignatureRequest` — a request for one or more parties to sign a document
- `Watermark` — a visual watermark overlay on a document
- `DigitalSignature` — cryptographic signature metadata
- `Document` — a document submitted for signature

---

## Query Patterns

### File operations
```graphql
query GetFileMetadata($path: String!) {
  file(path: $path) {
    ... on FileMetadata { id name size serverModified }
    ... on FolderMetadata { id name pathDisplay }
  }
}
```

### Search
```graphql
query SearchFiles($query: String!, $options: SearchOptionsInput) {
  search(query: $query, options: $options) {
    matches { metadata { ... on FileMetadata { name size } } matchType }
    hasMore cursor
  }
}
```

### Sharing
```graphql
query ListSharedFolders {
  sharedFolders { sharedFolderId name policy { acl { aclUpdatePolicy } } }
}
```

### Team members
```graphql
query ListTeamMembers {
  teamMembers {
    profile { accountId name { displayName } email }
    role { adminTier }
    status
  }
}
```
