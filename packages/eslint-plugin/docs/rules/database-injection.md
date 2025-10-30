# database-injection

Comprehensive database injection detection across SQL, NoSQL, and ORM queries (SonarQube-inspired).

⚠️ This rule **_errors_** in the `recommended` config.

## Rule Details

This advanced rule detects injection vulnerabilities across multiple database types:
- **SQL**: MySQL, PostgreSQL, SQLite, SQL Server
- **NoSQL**: MongoDB, Redis, DynamoDB  
- **ORMs**: Sequelize, TypeORM, Prisma (raw queries)

Inspired by SonarQube's comprehensive injection detection.

## Examples

### ❌ Incorrect

#### SQL Injection

```typescript
// Raw SQL with concatenation
db.query(`SELECT * FROM users WHERE email = '${email}'`);
pool.execute("DELETE FROM logs WHERE id = " + logId);

// Sequelize raw queries
sequelize.query(`SELECT * FROM users WHERE name = '${name}'`);
```

#### NoSQL Injection

```typescript
// MongoDB injection
db.collection('users').find({ username: req.body.username });
// If username is: { $gt: "" }, returns all users!

// Redis injection  
redis.eval(`return redis.call('GET', '${key}')`, 0);
```

#### ORM Misuse

```typescript
// TypeORM raw query
const users = await getRepository(User)
  .query(`SELECT * FROM users WHERE role = '${role}'`);

// Prisma raw query
await prisma.$executeRaw(`DELETE FROM users WHERE id = ${userId}`);
```

### ✅ Correct

#### SQL - Parameterized Queries

```typescript
// MySQL parameterized
db.query('SELECT * FROM users WHERE email = ?', [email]);
pool.execute('DELETE FROM logs WHERE id = ?', [logId]);

// PostgreSQL $1, $2 syntax
client.query('SELECT * FROM users WHERE email = $1', [email]);
```

#### NoSQL - Safe Queries

```typescript
// MongoDB safe query
db.collection('users').find({ username: { $eq: req.body.username } });

// Redis with validation
const safeKey = key.replace(/[^a-zA-Z0-9]/g, '');
redis.get(safeKey);
```

#### ORM - Query Builders

```typescript
// TypeORM query builder
const users = await getRepository(User)
  .createQueryBuilder('user')
  .where('user.role = :role', { role })
  .getMany();

// Prisma safe query
await prisma.user.delete({ where: { id: userId } });

// Sequelize parameterized
sequelize.query(
  'SELECT * FROM users WHERE name = ?',
  { replacements: [name], type: QueryTypes.SELECT }
);
```

## Configuration

```javascript
{
  rules: {
    '@forge-js/llm-optimized/database-injection': ['error', {
      detectNoSQL: true,         // Check MongoDB, Redis, etc.
      detectORMs: true,          // Check Sequelize, TypeORM raw queries
      trustedSources: [],        // Variables considered safe
      frameworkHints: true       // Show framework-specific suggestions
    }]
  }
}
```

## Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `detectNoSQL` | `boolean` | `true` | Detect NoSQL injection patterns (MongoDB, Redis) |
| `detectORMs` | `boolean` | `true` | Check ORM raw query methods |
| `trustedSources` | `string[]` | `[]` | Variable names considered safe (e.g., `CONFIG`, `SCHEMA`) |
| `frameworkHints` | `boolean` | `true` | Provide framework-specific fix suggestions |

### Disable NoSQL Detection

```javascript
{
  rules: {
    '@forge-js/llm-optimized/database-injection': ['error', {
      detectNoSQL: false,  // Only check SQL
      detectORMs: true
    }]
  }
}
```

### Trust Specific Variables

```javascript
{
  rules: {
    '@forge-js/llm-optimized/database-injection': ['error', {
      trustedSources: ['TABLES', 'COLUMNS', 'CONFIG']
    }]
  }
}
```

```typescript
// ✅ Won't flag trusted sources
const TABLES = { users: 'users', posts: 'posts' };
const query = `SELECT * FROM ${TABLES.users}`;
```

## Detected Patterns

### SQL Databases

- String concatenation in `query()`, `execute()`, `raw()`
- Template literals with variables
- `eval()` or `Function()` with SQL strings

### NoSQL Databases

- Direct object assignment from user input (MongoDB)
- `$where` operators with user strings
- Redis `eval()` with concatenation

### ORM Frameworks

- Sequelize `query()` with template literals
- TypeORM `query()` with interpolation
- Prisma `$executeRaw` with template literals

## Migration Guide

### From Raw SQL to Parameterized

```typescript
// Before
const results = await db.query(
  `SELECT * FROM users WHERE status = '${status}' AND role = '${role}'`
);

// After
const results = await db.query(
  'SELECT * FROM users WHERE status = ? AND role = ?',
  [status, role]
);
```

### From MongoDB Direct Assignment

```typescript
// Before
const user = await User.findOne({ 
  $where: `this.username == '${username}'`
});

// After
const user = await User.findOne({ 
  username: { $eq: username }
});
```

## Performance Impact

This rule performs deep AST analysis and may increase linting time:
- **Small projects (<100 files)**: Negligible (<100ms)
- **Medium projects (100-500 files)**: ~500ms
- **Large projects (>500 files)**: ~2s

Use `maxComplexity` option to limit analysis depth if needed.

## When Not To Use It

- Projects using only type-safe query builders (Prisma without raw queries)
- Migration scripts with hardcoded SQL
- Database seed files

Add these to `ignorePatterns`:

```javascript
{
  ignorePatterns: [
    '**/migrations/**',
    '**/seeds/**',
    '**/fixtures/**'
  ]
}
```

## Related Rules

- `no-sql-injection` - SQL-only detection (lighter weight)
- `no-unsafe-dynamic-require` - Dynamic require security

## Further Reading

- [SonarQube Injection Rules](https://rules.sonarsource.com/javascript/type/Vulnerability/RSPEC-2076)
- [OWASP NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection)
- [MongoDB Security Checklist](https://www.mongodb.com/docs/manual/administration/security-checklist/)

## Version

This rule is available in `@forge-js/eslint-plugin-llm-optimized` v0.0.1+

