# Hands on with PingIDM

## Prereqs

- Java 17
- Access to Ping software downloads

## Sample

- [IDM sample for bidirectional sync](https://docs.pingidentity.com/pingidm/7.5/samples-guide/sync-with-ldap-bidirectional.html)
- Run inbound reconciliation
- Update a user in IDM - observe outbound update to PingDS

## Add livesync

- Enable changelog numbers

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

([Watch the video](videos/livesync.mov))

## Inbound sync of groups

- Create mapping from LDAP group to organization
- LDAP `cn` -> `name`
- LDAP `description` -> `description`

Requires transformation for `cn` and `description` properties

```
source[0];
```

[Watch the video](videos/groupmapping.mov)

## Inbound sync of group members

- Add users to groups in LDAP
- Map LDAP `uniqueMember` property to organisation `members` property
- Transform to relationship

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

[Watch the video](videos/groupmembership.mov)
