# Claudate.com Skills Factory - Complete File Structure

Visual reference showing all files to be created in the Claudate.com project.

---

## 📁 Complete Project Structure

```
claudate-packages/                          # Your existing Claudate project root
│
├── apps/
│   ├── api/
│   │   ├── migrations/
│   │   │   ├── 001-013_*.sql              # Existing migrations
│   │   │   └── 014_add_user_generated_skills_support.sql  # 🆕 NEW
│   │   │
│   │   └── src/
│   │       ├── routes/
│   │       │   ├── packages.ts            # Existing
│   │       │   └── skills.ts              # 🆕 NEW (300 lines)
│   │       │
│   │       ├── services/
│   │       │   └── skill-generator/       # 🆕 NEW folder
│   │       │       ├── generator.ts       # 🆕 NEW (300 lines)
│   │       │       ├── packager.ts        # 🆕 NEW (50 lines)
│   │       │       └── templates/         # 🆕 NEW folder
│   │       │           ├── skill-md.ts    # 🆕 NEW (template)
│   │       │           ├── readme.ts      # 🆕 NEW (template)
│   │       │           ├── how-to-use.ts  # 🆕 NEW (template)
│   │       │           └── python-class.ts # 🆕 NEW (template)
│   │       │
│   │       └── validators/
│   │           └── skill-validator.ts     # 🆕 NEW (150 lines)
│   │
│   └── web/
│       └── src/
│           └── pages/
│               ├── skills/
│               │   ├── Create.tsx         # 🆕 NEW (250 lines)
│               │   ├── MySkills.tsx       # 🆕 NEW (100 lines)
│               │   └── [slug].tsx         # Existing (modify filter)
│               │
│               └── App.tsx                # Modify (add routes)
│
├── packages/
│   └── shared/
│       └── types/
│           └── skill.ts                   # 🆕 NEW (80 lines)
│
└── integration/                           # 🆕 NEW folder
    └── skills-factory-assets/             # 🆕 NEW folder
        ├── templates/
        │   ├── skill-templates.ts         # 🆕 EXTRACTED
        │   └── validation-rules.ts        # 🆕 EXTRACTED
        └── examples/
            └── example-skills.ts          # 🆕 EXTRACTED
```

---

## 📊 File Summary

### New Files Created: 14

| Category | File Count | Total Lines |
|----------|-----------|-------------|
| Migration | 1 | 15 |
| Services | 5 | ~400 |
| API Routes | 1 | ~300 |
| Validators | 1 | ~150 |
| Frontend | 2 | ~350 |
| Types | 1 | ~80 |
| Assets | 3 | ~200 |
| **Total** | **14** | **~1,500** |

### Modified Files: 2

| File | Changes |
|------|---------|
| `apps/web/src/pages/App.tsx` | Add 2 new routes |
| `apps/web/src/pages/skills/[slug].tsx` | Add filter option |

---

## 🗄️ Database Changes

### Current Structure
```sql
packages (16 columns)
- id, type, name, slug, display_name, description_short
- source_type, author_name, readme_key, metadata_key
- downloads_count, stars_count, status
- created_at, updated_at
- [other fields...]
```

### After Migration 014
```sql
packages (17 columns)  ← Only +1 column!
- [all previous columns]
- creator_user_id       # 🆕 NEW: Links to Clerk user ID
```

**Indexes Added**:
- `idx_packages_creator` on `creator_user_id`
- `idx_packages_source_type` on `source_type`

---

## 🔌 API Endpoints

### New Endpoints (4 total)

```
POST   /api/skills/generate       # Create new skill
GET    /api/skills/:id/download   # Download skill ZIP
GET    /api/skills/my-skills      # List user's skills
DELETE /api/skills/:id            # Delete skill
```

### Request/Response Examples

#### POST /generate
```typescript
// Request
{
  name: "data-analyzer",
  displayName: "Data Analyzer",
  description: "Analyze CSV and JSON data files",
  category: "data-analysis",
  capabilities: ["Parse CSV", "Parse JSON", "Generate stats"],
  useCases: ["Analyze sales data", "Process logs"],
  inputRequirements: "CSV or JSON file",
  outputFormats: "Summary report",
  includePython: true
}

// Response
{
  success: true,
  skill: {
    id: "abc123",
    name: "data-analyzer",
    displayName: "Data Analyzer",
    version: "1.0.0",
    downloadUrl: "/api/skills/abc123/download",
    viewUrl: "/skills/user-12ab-data-analyzer"
  },
  metadata: {
    name: "data-analyzer",
    displayName: "Data Analyzer",
    version: "1.0.0",
    createdAt: "2025-11-10T10:30:00Z",
    fileCount: 6,
    totalSize: 15234
  }
}
```

---

## 🎨 Frontend Routes

### New Routes (2 total)

```
/skills/create          → Create.tsx (multi-step wizard)
/skills/my-skills       → MySkills.tsx (user dashboard)
```

### Modified Routes (1)

```
/skills/:slug           → Add "User Generated" filter tab
```

### Navigation Changes

```
Header Navigation
├── Skills (existing)
├── Create Skill (new)  ← Add this link
└── My Skills (new)     ← Add this link
```

---

## 🎯 Component Hierarchy

### Create Skill Page

```
Create.tsx
├── ProgressSteps (5 steps)
│   ├── Step 1: Basic Info
│   │   ├── NameInput (kebab-case validation)
│   │   ├── DisplayNameInput
│   │   ├── DescriptionTextarea (20-200 chars)
│   │   └── CategorySelect
│   │
│   ├── Step 2: Capabilities
│   │   └── CapabilityList (3-10 items)
│   │       └── CapabilityInput (add/remove)
│   │
│   ├── Step 3: Usage
│   │   ├── InputRequirementsTextarea
│   │   ├── OutputFormatsTextarea
│   │   └── UseCasesList (1-5 items)
│   │
│   ├── Step 4: Options
│   │   ├── IncludePythonCheckbox
│   │   └── PythonOptionsPanel (conditional)
│   │
│   └── Step 5: Review
│       ├── SummaryCard
│       ├── ErrorDisplay (if validation fails)
│       └── GenerateButton
│
└── NavigationButtons
    ├── PreviousButton
    └── NextButton (or GenerateButton on step 5)
```

### My Skills Dashboard

```
MySkills.tsx
├── Header
│   ├── Title ("My Skills")
│   └── CreateButton → /skills/create
│
├── SkillsList (or EmptyState)
│   └── SkillCard (for each skill)
│       ├── SkillTitle
│       ├── SkillDescription
│       ├── SkillMeta (downloads, date)
│       └── Actions
│           ├── DownloadButton
│           ├── ViewButton
│           └── DeleteButton
│
└── Pagination (if >10 skills)
```

---

## 🔄 Data Flow

### Skill Creation Flow

```
User Form Input
    ↓
Frontend Validation (React)
    ↓
POST /api/skills/generate
    ↓
Backend Validation (SkillValidator)
    ↓
Skill Generation (SkillGenerator)
    ↓
ZIP Creation (SkillPackager)
    ↓
R2 Upload (3 files)
    - README.md
    - metadata.json
    - skill-name.zip
    ↓
D1 Insert (packages table)
    - type: 'skill'
    - source_type: 'user-generated'
    - creator_user_id: userId
    - readme_key: R2 path
    - metadata_key: R2 path
    ↓
D1 Insert (package_versions table)
    - version: '1.0.0'
    - download_key: R2 path to ZIP
    ↓
Response to User
    ↓
Redirect to Skill Page
```

### Skill Download Flow

```
User Clicks "Download"
    ↓
GET /api/skills/:id/download
    ↓
Fetch from D1 (packages + package_versions)
    ↓
Fetch from R2 (ZIP file)
    ↓
Increment downloads_count
    ↓
Stream ZIP to Browser
    ↓
Browser Downloads ZIP File
```

---

## 📦 Dependencies to Add

### Backend (package.json)

```json
{
  "dependencies": {
    "mustache": "^4.2.0",      // Template engine
    "fflate": "^0.8.1"         // ZIP compression
  }
}
```

### Frontend (package.json)

```json
{
  "dependencies": {
    // All already installed in typical React project
    "react": "^18.x",
    "react-router-dom": "^6.x",
    "@clerk/clerk-react": "^4.x"
  }
}
```

**Total New Dependencies**: 2 (both lightweight)

---

## 🧪 Testing Strategy

### Unit Tests

```
apps/api/src/services/skill-generator/__tests__/
├── generator.test.ts          # Test skill generation logic
├── packager.test.ts           # Test ZIP creation
└── validator.test.ts          # Test validation rules
```

### Integration Tests

```
apps/api/src/routes/__tests__/
└── skills.test.ts             # Test API endpoints
```

### E2E Tests

```
apps/web/e2e/
└── skill-creation.spec.ts     # Test complete user flow
```

---

## 📝 Configuration Files

### TypeScript Config (No Changes Needed)

Existing `tsconfig.json` should work as-is. Verify these settings:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "types": ["@cloudflare/workers-types"]
  }
}
```

### Environment Variables (Add to .dev.vars)

```bash
# No new environment variables needed!
# Uses existing:
# - DATABASE binding (D1)
# - R2 binding
# - Clerk auth
```

---

## 🚀 Deployment Steps

### 1. Database Migration

```bash
# Local test
wrangler d1 execute claudate-packages-db --local \
  --file=apps/api/migrations/014_add_user_generated_skills_support.sql

# Production deployment
wrangler d1 execute claudate-packages-db-prod \
  --file=apps/api/migrations/014_add_user_generated_skills_support.sql
```

### 2. Backend Deployment

```bash
cd apps/api
npm run build
wrangler deploy
```

### 3. Frontend Deployment

```bash
cd apps/web
npm run build
wrangler pages deploy dist
```

---

## 📊 Size Analysis

### Backend Bundle

| Component | Size (gzipped) |
|-----------|---------------|
| SkillGenerator | ~15 KB |
| SkillPackager | ~10 KB |
| Templates | ~5 KB |
| Validators | ~3 KB |
| API Routes | ~8 KB |
| **Total** | **~41 KB** |

### Frontend Bundle

| Component | Size (gzipped) |
|-----------|---------------|
| Create Page | ~25 KB |
| My Skills Page | ~10 KB |
| Shared Components | ~5 KB |
| **Total** | **~40 KB** |

**Total Integration Impact**: ~81 KB (negligible for modern workers)

---

## ✅ Verification Checklist

After implementation, verify:

### Database
- [ ] `creator_user_id` field exists in packages table
- [ ] Indexes created (creator, source_type)
- [ ] Can insert user-generated skill records
- [ ] Can query by creator_user_id
- [ ] Can filter by source_type

### Backend
- [ ] SkillValidator validates correctly
- [ ] SkillGenerator creates complete packages
- [ ] SkillPackager creates valid ZIPs
- [ ] API endpoints return correct responses
- [ ] Authentication middleware works
- [ ] R2 uploads succeed

### Frontend
- [ ] Multi-step form navigates correctly
- [ ] Form validation works
- [ ] Skill generation triggers successfully
- [ ] Download button works
- [ ] My Skills dashboard displays correctly
- [ ] Delete function works
- [ ] Navigation links added

### Integration
- [ ] End-to-end skill creation works
- [ ] ZIP downloads correctly
- [ ] Skills appear in marketplace
- [ ] Filters work (Official vs User Generated)
- [ ] User can manage their skills

---

## 📚 Next Steps

1. **Verify database**: Run `PRAGMA table_info(packages)`
2. **Start extraction**: Extract templates from open source
3. **Implement services**: Create TypeScript services
4. **Build API**: Implement routes and endpoints
5. **Create UI**: Build multi-step form and dashboard
6. **Test**: Run end-to-end tests
7. **Deploy**: Push to production
8. **Launch**: Announce feature to users

---

**Document Version**: 1.0
**Last Updated**: 2025-11-10
**Status**: Ready for implementation

**Visual reference complete!** Use this alongside the integration package for implementation.
