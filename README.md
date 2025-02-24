# Hands on with PingIDM

## Prereqs

- Java 17
- Access to Ping software downloads

## Sample

- [IDM sample for bidirectional sync](https://docs.pingidentity.com/pingidm/7.5/samples-guide/sync-with-ldap-bidirectional.html)

## Add group sync

- Create new managed object `group`
- Properties `name`, `description`
- Create mapping from LDAP group to new object
- LDAP `cn` -> `name`

Requires transformation for `cn` property

```
source[0];
```

## Sync group members

- Add `members` property to `group` object
- Two way relationship with `user`
- Add users to groups in LDAP
- Map LDAP `uniqueMember` property to user `members` property
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
