# MongoDB v7 Compatibility

## Overview

This is a fork of [strongloop/loopback-connector-mongodb](https://github.com/strongloop/loopback-connector-mongodb) v4.2.0, modified to support MongoDB v7 and v8 server versions while maintaining backward compatibility with MongoDB v6.

**Fork maintained by:** Hebeworks
**Base version:** v4.2.0
**Driver upgrade:** MongoDB Node.js driver v3.1.4 → v4.17.2

## Why This Fork Exists

The official `loopback-connector-mongodb` v4.2.0 uses MongoDB Node.js driver v3.x, which does not support connections to MongoDB server v7 and v8 due to wire protocol incompatibilities. This fork upgrades the MongoDB driver to v4.17.2, enabling compatibility with modern MongoDB servers while preserving the LoopBack v3 connector API.

## Wire Protocol Compatibility Matrix

| Connector Version | MongoDB Driver | MongoDB Server Support |
|------------------|----------------|------------------------|
| Official v4.2.0 | v3.1.4 | v3.6, v4.0, v4.2, v4.4, v5.0, v6.0 |
| **This Fork** | **v4.17.2** | **v6.0, v7.0, v8.0** |

## Changes From Original

### 1. MongoDB Driver Upgrade
- **Driver version:** `^3.1.4` → `^4.17.2`
- **Reason:** Enable MongoDB v7/v8 wire protocol support

### 2. Cursor Handling for `find` and `aggregate`
MongoDB driver v4 changed how cursor-returning operations work:
- **Driver v3:** `find()` and `aggregate()` accept callbacks and return results
- **Driver v4:** `find()` and `aggregate()` return cursor objects without callbacks

**Fix applied:**
```javascript
// Commands that return cursors in driver v4
const cursorCommands = ['find', 'aggregate'];

// In execute method:
if (cursorCommands.indexOf(command) >= 0) {
  // Remove callback from args for cursor commands
  args = args.slice(0, -1);
  var cursor = collection[command].apply(collection, args);
  return resultCallback(null, cursor);
} else {
  // For non-cursor commands, add callback to args
  args[args.length - 1] = resultCallback;
  return collection[command].apply(collection, args);
}
```

The connector's `all()` method already handles cursors correctly by calling `.toArray()`, so returning the cursor directly maintains compatibility.

### 3. Backward Compatibility Preserved

MongoDB driver v4.17.2 maintains v3 backward compatibility for callback-based operations:
- Returns both v3-style properties (`.ops`, `.result.n`, `.result.nModified`)
- Also returns v4-style properties (`.insertedId`, `.matchedCount`, `.modifiedCount`)

This means insert, update, and delete operations work without modification.

## Installation

### Via Git URL (Recommended)
```json
{
  "dependencies": {
    "loopback-connector-mongodb": "git+https://github.com/hebeworks/loopback-connector-mongodb.git#mongodb-v7-compatibility"
  }
}
```

Then run:
```bash
npm install
```

### Local Development
```bash
git clone https://github.com/hebeworks/loopback-connector-mongodb.git
cd loopback-connector-mongodb
git checkout mongodb-v7-compatibility
npm install
npm link

# In your application
npm link loopback-connector-mongodb
```

## Testing Strategy

### Unit Tests
Run the connector's built-in test suite:
```bash
npm test
```

### Integration Testing Against Multiple MongoDB Versions
1. Use Docker Compose to run MongoDB v6, v7, and v8
2. Test all CRUD operations against each version
3. Validate role initialization and complex queries

Example docker-compose.yml:
```yaml
services:
  mongodb-v6:
    image: mongo:6
    ports:
      - "27117:27017"

  mongodb-v7:
    image: mongo:7
    ports:
      - "27217:27017"

  mongodb-v8:
    image: mongo:8
    ports:
      - "27317:27017"
```

## Known Limitations

1. **LoopBack v3 Only:** This fork is designed for LoopBack v3 applications. LoopBack v4 uses a different connector architecture.

2. **Driver v4 Features:** This fork uses driver v4 with callback APIs for backward compatibility. Modern promise/async-await patterns are not utilized.

3. **Minimal Changes Philosophy:** Only essential changes were made. No refactoring or optimization beyond MongoDB v7 support.

## Migration From Official Connector

### Step 1: Update package.json
```diff
{
  "dependencies": {
-   "loopback-connector-mongodb": "^4.2.0",
+   "loopback-connector-mongodb": "git+https://github.com/hebeworks/loopback-connector-mongodb.git#mongodb-v7-compatibility"
  }
}
```

### Step 2: Install
```bash
npm install
```

### Step 3: Test
No code changes required in your application. Test thoroughly:
```bash
npm test
npm start
```

### Step 4: Validate
- Verify database connections succeed
- Test all CRUD operations
- Check complex queries and aggregations
- Validate any custom MongoDB operations

## Rollback Plan

If issues arise, revert to the official connector:

```diff
{
  "dependencies": {
-   "loopback-connector-mongodb": "git+https://github.com/hebeworks/loopback-connector-mongodb.git#mongodb-v7-compatibility",
+   "loopback-connector-mongodb": "^4.2.0"
  }
}
```

Then:
```bash
rm -rf node_modules/loopback-connector-mongodb
npm install
```

**Note:** You'll need to downgrade your MongoDB server to v6 or below if rolling back.

## Maintenance

This fork will be maintained for MongoDB v7/v8 compatibility needs. Updates will be minimal and focused on:
- Security patches from upstream MongoDB driver
- Critical bug fixes
- MongoDB server compatibility updates

## Contributing

Issues and pull requests welcome at: https://github.com/hebeworks/loopback-connector-mongodb

## License

MIT (same as original strongloop/loopback-connector-mongodb)

## Acknowledgments

- Original connector by StrongLoop/IBM
- MongoDB driver team for v4.17.2 backward compatibility
- LoopBack community for v3 support
