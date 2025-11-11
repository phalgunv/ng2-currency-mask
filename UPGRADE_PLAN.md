# Angular 14-20 Upgrade Plan for ng2-currency-mask

## Overview

This document outlines the strategy to progressively upgrade the ng2-currency-mask library from Angular 13 to support Angular versions 14 through 20, while maintaining backward compatibility and API stability.

## Goals

1. **Maintain API Stability**: Keep the public API unchanged to avoid breaking changes for consumers
2. **Progressive Compatibility**: Support Angular 13-20 with a single build using permissive peer dependencies
3. **Automated Testing**: Implement CI matrix testing across all supported Angular versions
4. **Type Safety**: Ensure TypeScript compatibility across all versions
5. **Ivy Compatibility**: Maintain partial compilation mode for optimal compatibility

## Key Files & Components

### Library Files
- `projects/ng2-currency-mask/package.json` - Library package configuration
- `projects/ng2-currency-mask/src/public-api.ts` - Public API surface
- `projects/ng2-currency-mask/src/lib/currency-mask.module.ts` - CurrencyMaskModule
- `projects/ng2-currency-mask/src/lib/currency-mask.directive.ts` - CurrencyMaskDirective
- `projects/ng2-currency-mask/src/lib/currency-mask.config.ts` - Configuration injection token
- `projects/ng2-currency-mask/ng-package.json` - ng-packagr configuration
- `projects/ng2-currency-mask/tsconfig.lib.prod.json` - Production TypeScript config

### Test Application Files
- `package.json` - Root package with devDependencies
- `projects/library-test/` - Example application for testing
- `angular.json` - Workspace configuration

## Step-by-Step Implementation Plan

### Phase 1: Preparation & Branch Strategy

#### 1.1 Create Feature Branches
Create dedicated branches for testing each Angular version:

```bash
git checkout -b upgrade/ang-14
git checkout -b upgrade/ang-15
git checkout -b upgrade/ang-16
git checkout -b upgrade/ang-17
git checkout -b upgrade/ang-18
git checkout -b upgrade/ang-19
git checkout -b upgrade/ang-20
```

Keep `master` branch stable during testing.

#### 1.2 Review Current State
- Document current API surface
- Identify all public exports in `public-api.ts`
- List any deprecated Angular APIs currently in use
- Verify current TypeScript version compatibility (4.5.2)

### Phase 2: Update Peer Dependencies

#### 2.1 Update Library Package
Edit `projects/ng2-currency-mask/package.json` to support Angular 13-20:

```json
{
  "peerDependencies": {
    "@angular/common": ">=13.0.0 <21.0.0",
    "@angular/core": ">=13.0.0 <21.0.0",
    "@angular/forms": ">=13.0.0 <21.0.0"
  }
}
```

**Rationale**: 
- Include `@angular/forms` explicitly (used by the directive)
- Use range syntax to support all versions 13-20
- Upper bound at 21 to be explicit about tested versions

#### 2.2 Update Development Dependencies
For each Angular version, align compatible TypeScript versions:

| Angular Version | TypeScript Support |
|----------------|-------------------|
| 13 | 4.4 - 4.5 |
| 14 | 4.6 - 4.8 |
| 15 | 4.8 - 4.9 |
| 16 | 4.9 - 5.0 |
| 17 | 5.2 - 5.3 |
| 18 | 5.4 - 5.5 |
| 19 | 5.5 - 5.6 |
| 20 | 5.6+ |

### Phase 3: Continuous Integration Setup

#### 3.1 Create GitHub Actions Workflow
Create `.github/workflows/ci.yml`:

```yaml
name: CI Matrix Build

on:
  push:
    branches: [ master, 'upgrade/**' ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    name: Build & Test (Angular ${{ matrix.angular }}, Node ${{ matrix.node }})
    runs-on: ubuntu-latest
    
    strategy:
      fail-fast: false
      matrix:
        angular: [13, 14, 15, 16, 17, 18, 19, 20]
        node: [18, 20]
        exclude:
          # Angular 13-15 don't support Node 20
          - angular: 13
            node: 20
          - angular: 14
            node: 20
          - angular: 15
            node: 20
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Update Angular to version ${{ matrix.angular }}
        run: |
          npx -y npm-check-updates '/^@angular\//' -t minor -u
          npm install
      
      - name: Build library
        run: npm run build --if-present
      
      - name: Lint
        run: npm run lint --if-present
      
      - name: Build test application
        run: npx ng build library-test --configuration development
```

#### 3.2 Add Build Status Badge
Update `README.md` to include CI status badge.

### Phase 4: Per-Version Testing & Fixes

For each Angular version (14-20), follow this process:

#### 4.1 Test Angular 14

```bash
# Switch to upgrade branch
git checkout upgrade/ang-14

# Update Angular CLI and Core
npx ng update @angular/core@14 @angular/cli@14 --force

# Update TypeScript to compatible version (4.6-4.8)
npm install --save-dev typescript@~4.8.0

# Install dependencies
npm ci

# Build library
npm run build

# Build test application
npx ng build library-test

# Run development server for manual testing
npm start
```

**Expected Changes**:
- Minor TypeScript type strictness improvements
- No breaking changes in ControlValueAccessor interface
- Ivy compiler stable

**Potential Issues**:
- Check for any deprecated APIs warnings
- Verify form control value accessor typing

#### 4.2 Test Angular 15

```bash
git checkout upgrade/ang-15
npx ng update @angular/core@15 @angular/cli@15 --force
npm install --save-dev typescript@~4.9.0
npm ci
npm run build
```

**Expected Changes**:
- Standalone components introduced (optional, don't need to adopt)
- Functional guards (not applicable to library)
- Improved TypeScript types

**Potential Issues**:
- Check if any internal APIs changed
- Verify injection token types

#### 4.3 Test Angular 16

```bash
git checkout upgrade/ang-16
npx ng update @angular/core@16 @angular/cli@16 --force
npm install --save-dev typescript@~5.0.0
npm ci
npm run build
```

**Expected Changes**:
- Signals introduced (optional)
- Required inputs (not applicable)
- TypeScript 5.0 support

**Potential Issues**:
- TypeScript 5.0 breaking changes in strict mode
- Check esbuild compatibility with ng-packagr

#### 4.4 Test Angular 17

```bash
git checkout upgrade/ang-17
npx ng update @angular/core@17 @angular/cli@17 --force
npm install --save-dev typescript@~5.3.0
npm ci
npm run build
```

**Expected Changes**:
- New control flow syntax (doesn't affect library)
- Vite/esbuild default (test app only)
- Deferred loading (doesn't affect library)

**Potential Issues**:
- Ensure ng-packagr is compatible with Angular 17
- May need to update ng-packagr to ^17.0.0

#### 4.5 Test Angular 18

```bash
git checkout upgrade/ang-18
npx ng update @angular/core@18 @angular/cli@18 --force
npm install --save-dev typescript@~5.5.0
npm ci
npm run build
```

**Expected Changes**:
- Zoneless support (optional)
- Material 3 (not applicable)
- Enhanced TypeScript support

**Potential Issues**:
- Check for any form control API changes
- Verify zone.js compatibility

#### 4.6 Test Angular 19

```bash
git checkout upgrade/ang-19
npx ng update @angular/core@19 @angular/cli@19 --force
npm install --save-dev typescript@~5.6.0
npm ci
npm run build
```

**Expected Changes**:
- Incremental hydration improvements (not applicable)
- Enhanced standalone APIs

**Potential Issues**:
- Verify ng-packagr compatibility
- Check for any breaking changes in forms

#### 4.7 Test Angular 20

```bash
git checkout upgrade/ang-20
npx ng update @angular/core@20 @angular/cli@20 --force
npm install --save-dev typescript@~5.6.0
npm ci
npm run build
```

**Expected Changes**:
- Latest features and improvements

**Potential Issues**:
- May require latest ng-packagr
- Check for any deprecated API removals

### Phase 5: Code Updates & Compatibility Fixes

#### 5.1 Common Areas to Review

**CurrencyMaskDirective (`currency-mask.directive.ts`)**:
- ControlValueAccessor interface implementation
- Validator interface (if used)
- Host listener decorators
- Element injection
- Type annotations for callbacks (onChange, onTouched)

```typescript
// Ensure proper typing for callbacks
private onChange = (value: any) => {};
private onTouched = () => {};

writeValue(value: any): void { /* ... */ }
registerOnChange(fn: any): void { this.onChange = fn; }
registerOnTouched(fn: any): void { this.onTouched = fn; }
setDisabledState?(isDisabled: boolean): void { /* ... */ }
```

**InputManager (`input.manager.ts`)**:
- DOM manipulation code
- Selection API usage
- Browser compatibility

**InputService (`input.service.ts`)**:
- Injectable decorator usage
- Provider configuration

**CurrencyMaskModule (`currency-mask.module.ts`)**:
- NgModule metadata
- Provider configuration
- InjectionToken usage

#### 5.2 TypeScript Configuration

Ensure `tsconfig.lib.prod.json` has correct settings:

```json
{
  "extends": "./tsconfig.lib.json",
  "compilerOptions": {
    "declarationMap": false
  },
  "angularCompilerOptions": {
    "compilationMode": "partial"
  }
}
```

**Key setting**: `"compilationMode": "partial"` enables Ivy partial compilation for better compatibility.

#### 5.3 ng-packagr Configuration

Verify `ng-package.json`:

```json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/ng2-currency-mask",
  "lib": {
    "entryFile": "src/public-api.ts"
  }
}
```

### Phase 6: Testing Strategy

#### 6.1 Automated Testing
- CI matrix builds across all Angular versions
- Lint checks
- Build verification

#### 6.2 Manual Testing Checklist
For each Angular version, test the example app:

- [ ] Currency input accepts numeric input
- [ ] Formatting applies correctly on blur
- [ ] Model binding works (ngModel)
- [ ] Reactive forms work (formControl)
- [ ] Prefix/suffix configuration works
- [ ] Decimal precision configuration works
- [ ] Thousands separator configuration works
- [ ] Allow negative values works
- [ ] Max/min value constraints work
- [ ] Disabled state works
- [ ] Selection/cursor position maintained

#### 6.3 Browser Testing
Test in:
- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)

### Phase 7: Version & Release Strategy

#### 7.1 Versioning Approach

**Option A: Single Version with Wide Peer Dependencies** (Recommended)
- Use peer dependencies `>=13.0.0 <21.0.0`
- Publish single version (e.g., v14.0.0) that works with all Angular versions
- Simpler maintenance

**Option B: Version per Angular Major**
- Publish v14.x for Angular 14
- Publish v15.x for Angular 15
- etc.
- More complex maintenance

**Recommendation**: Use Option A unless breaking changes are required.

#### 7.2 Release Checklist

Before releasing:
- [ ] All CI matrix tests pass
- [ ] Manual testing completed for all versions
- [ ] CHANGELOG.md updated
- [ ] Version bumped in package.json
- [ ] README.md updated with supported versions
- [ ] Git tag created
- [ ] npm publish executed
- [ ] GitHub release created

#### 7.3 Semantic Versioning

- **Patch** (x.x.X): Bug fixes, TypeScript type improvements
- **Minor** (x.X.0): New features, expanded Angular version support
- **Major** (X.0.0): Breaking API changes (avoid if possible)

### Phase 8: Documentation Updates

#### 8.1 README.md Updates
Add compatibility table:

```markdown
## Angular Version Compatibility

| ng2-currency-mask | Angular | TypeScript |
|-------------------|---------|------------|
| 13.x              | 13      | 4.5        |
| 14.x              | 13-20   | 4.5-5.6    |
```

#### 8.2 Migration Guide
Create `MIGRATION.md` if any changes needed.

#### 8.3 API Documentation
Ensure all public APIs documented in README.

### Phase 9: Maintenance & Automation

#### 9.1 Dependabot Configuration
Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      angular:
        patterns:
          - "@angular/*"
      typescript:
        patterns:
          - "typescript"
```

#### 9.2 Automated Dependency Updates
- Configure Renovate or Dependabot
- Group Angular packages together
- Auto-merge patch updates after CI passes

#### 9.3 Regular Testing
- Run CI weekly even without changes
- Test against Angular beta/rc versions before major releases

## Risk Assessment

### Low Risk
- Angular 13-15: Minimal changes, stable Ivy
- TypeScript updates: Mostly additive features

### Medium Risk
- Angular 16-17: TypeScript 5.0 breaking changes
- ng-packagr compatibility across versions

### High Risk
- Angular 18-20: Potential for new deprecations
- Future form API changes

## Rollback Plan

If issues arise:
1. Revert peer dependency changes
2. Publish patch version with stricter version ranges
3. Document issues in GitHub
4. Fix and re-release

## Success Criteria

- [ ] Library builds successfully with Angular 13-20
- [ ] All CI tests pass
- [ ] No runtime errors in test application
- [ ] Performance remains unchanged
- [ ] Bundle size doesn't increase significantly
- [ ] No breaking changes to public API
- [ ] Documentation updated
- [ ] Published to npm

## Timeline Estimate

- **Phase 1-2**: 1 day (Preparation & peer deps)
- **Phase 3**: 1 day (CI setup)
- **Phase 4-5**: 3-5 days (Testing & fixes per version)
- **Phase 6**: 2 days (Comprehensive testing)
- **Phase 7-8**: 1 day (Release & docs)
- **Phase 9**: 1 day (Automation setup)

**Total**: ~2 weeks for complete implementation and testing

## Quick Reference Commands

```bash
# Create branch
git checkout -b upgrade/ang-14

# Update to specific Angular version
npx ng update @angular/core@14 @angular/cli@14 --force

# Update TypeScript
npm install --save-dev typescript@~4.8.0

# Install dependencies
npm ci

# Build library
npm run build

# Build test app
npx ng build library-test

# Run dev server
npm start

# Lint
npm run lint

# Check for outdated packages
npx npm-check-updates

# Update all Angular packages
npx npm-check-updates '/^@angular\//' -u
```

## Resources

- [Angular Update Guide](https://update.angular.io/)
- [Angular Versions](https://angular.io/guide/versions)
- [TypeScript Compatibility](https://www.typescriptlang.com/docs/handbook/release-notes/overview.html)
- [ng-packagr Documentation](https://github.com/ng-packagr/ng-packagr)
- [Ivy Compatibility](https://angular.io/guide/ivy-compatibility)

## Contact & Support

For questions or issues during the upgrade process:
- Create GitHub issue
- Reference this plan document
- Include error logs and version information
