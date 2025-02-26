# Hands on with PingIDM

## Prereqs

- Java 17
- Access to Ping software downloads

## Sample

- [IDM sample for bidirectional sync](https://docs.pingidentity.com/pingidm/7.5/samples-guide/sync-with-ldap-bidirectional.html)
- Install and start PingDS
- Install and start PingIDM with sample profile
- Explore connector and mapping in IDM admin console
- Run inbound reconciliation - observe users created
- Update a user in IDM - observe outbound update to PingDS

## Add livesync

- Enable changelog with change numbers

```
ds/bin/dsconfig set-replication-server-prop \
          --provider-name "Multimaster Synchronization" \
          --set changelog-enabled:enabled \
          --hostname localhost \
          --port 4444 \
          --bindDn uid=admin \
          --bindPassword password \
          --trustAll \
          --no-prompt
```

(or use dsconfig interactively - [watch the video](videos/changelog.mov))

- Update connector with livesync base DN `dc=com`
- Add a schedule for livesync at 30s interval
- Change in PingDS - observe update in IDM

[Watch the video](videos/livesync.mov)

## Inbound sync of roles

- Create mapping from LDAP groups to IDM roles
- LDAP `cn` -> IDM `name`
- LDAP `description` -> IDM `description`

Requires transformation for `cn` and `description` properties as follows (because LDAP is configured with multivalue properties)

```
source[0];
```

[Watch the video](videos/rolemapping.mov)

## Inbound sync of group members to role members

- Add users to groups in LDAP
- Map LDAP `uniqueMember` property to role `members` property
- Transform to an IDM relationship as follows:

```
function extractUID(dn) {
    const match = dn.match(/uid=([^,]+)/);
    return match ? match[1] : null;
}

var members = [];
source.forEach((uniqueMember) => {
  const uid = extractUID(uniqueMember);
  if (!uid) {
    return;
  }

  const queryFilter = `userName eq "${extractUID(uniqueMember)}"`;
  const searchResults = openidm.query("managed/user", {"_queryFilter": queryFilter }, ["_id"]);
  if (!searchResults || searchResults.result.length !== 1) {
    return;
  }

  members.push({ _ref : `managed/user/${searchResults.result[0]._id}` });
});

members;
```

[Watch the video](videos/rolemembers.mov)
