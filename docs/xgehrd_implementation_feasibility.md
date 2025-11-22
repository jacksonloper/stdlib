<!--

@license Apache-2.0

Copyright (c) 2025 The Stdlib Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

-->

# xGEHRD Implementation Feasibility Assessment

> Feasibility assessment for implementing xGEHRD (Hessenberg reduction) routines in stdlib.

## Overview

This document assesses the feasibility of implementing the xGEHRD family of LAPACK routines in stdlib. xGEHRD reduces a general matrix to upper Hessenberg form using orthogonal similarity transformations, which is a key step in eigenvalue computation algorithms.

## What is xGEHRD?

The xGEHRD family consists of:
- **SGEHRD**: Single-precision real
- **DGEHRD**: Double-precision real
- **CGEHRD**: Single-precision complex
- **ZGEHRD**: Double-precision complex

These routines reduce a general n-by-n matrix A to upper Hessenberg form H by an orthogonal/unitary similarity transformation: Q^T * A * Q = H (or Q^H * A * Q = H for complex).

## Developer Guidelines Summary

According to stdlib's developer guidelines (`docs/contributing/packages.md`), implementing a new package involves:

### Required Package Structure

Every package must contain:

1. **benchmark/** - Performance benchmarks
2. **docs/** - Documentation (excluding README)
   - repl.txt (REQUIRED)
   - usage.txt
3. **examples/** - Usage examples
   - index.js (main example matching README)
4. **lib/** - Package implementation
   - index.js (main exports)
   - main.js (typical implementation file)
   - base.js (base JavaScript implementation)
   - native.js (native add-on wrapper, if applicable)
   - ndarray.js (ndarray interface)
   - validate.js (input validation)
5. **test/** - Unit tests (100% coverage required)
6. **package.json** - Package metadata
7. **README.md** - Main documentation

### Additional Components for LAPACK Packages

Based on existing LAPACK packages (e.g., dlacpy, dlaset):

8. **src/** - Source files requiring compilation
   - addon.c (Node.js native add-on binding)
   - [routine].c (C implementation)
   - [routine]_ndarray.c (ndarray C implementation)
   - Makefile
9. **include/** - C header files
   - stdlib/lapack/base/[routine].h
10. **binding.gyp** - GYP file for native add-ons
11. **include.gypi** - GYP include file
12. **manifest.json** - Native add-on metadata
13. **docs/types/** - TypeScript declarations
    - index.d.ts
    - test.ts

## Implementation Requirements by Development Phase

### Phase 1: README-First Development

**Effort**: Medium  
**Artifacts**: README.md

- Define API signature: `xgehrd( order, n, ilo, ihi, A, lda, tau )`
- Document parameters:
  - order: storage layout ('row-major' or 'column-major')
  - n: order of matrix A
  - ilo, ihi: indices defining the active submatrix
  - A: input/output matrix (Float64Array for DGEHRD)
  - lda: leading dimension of A
  - tau: output array storing scalar factors of elementary reflectors
- Provide usage examples
- Document mathematical background (Hessenberg form, Householder reflections)
- Include references to LAPACK documentation

### Phase 2: Package Metadata

**Effort**: Low  
**Artifacts**: package.json

- Package name: @stdlib/lapack/base/[d|s|c|z]gehrd
- Description: "Reduce a general matrix to upper Hessenberg form"
- Keywords: stdlib, lapack, gehrd, hessenberg, linear algebra, eigenvalues, householder
- Mark as having native add-on: "gypfile": true

### Phase 3: Examples

**Effort**: Low-Medium  
**Artifacts**: examples/index.js, examples/*.js

- Create main example demonstrating Hessenberg reduction
- Show how to work with different matrix layouts
- Demonstrate ilo/ihi usage for partial reduction
- Example with result verification

### Phase 4: Implementation

**Effort**: High  
**Artifacts**: lib/*.js, src/*.c, include/*.h

This is the most complex phase requiring:

#### JavaScript Implementation (lib/)

1. **lib/main.js**: Wrapper selecting between native and JS implementations
2. **lib/base.js**: Pure JavaScript implementation (~200-400 lines estimated)
   - Implement Householder QR-like algorithm
   - Handle row-major and column-major layouts
   - Complex loop structure for computing and applying Householder reflectors
3. **lib/native.js**: Native add-on wrapper
4. **lib/ndarray.js**: Alternative indexing semantics
5. **lib/ndarray.native.js**: Native version of ndarray interface
6. **lib/index.js**: Main entry point with conditional exports
7. **lib/validate.js**: Input validation (~100-150 lines)

#### C/Fortran Implementation (src/)

1. **src/addon.c**: Node.js N-API binding (~100-150 lines)
2. **src/dgehrd.c**: C wrapper calling LAPACK (~50-100 lines)
3. **src/dgehrd_ndarray.c**: C implementation with ndarray semantics (~200-300 lines)

**Key Challenge**: Need to either:
- Link against external LAPACK library (BLAS/LAPACK dependency)
- Port reference Fortran DGEHRD implementation (~500+ lines from Netlib)
- Implement algorithm from scratch in C

GEHRD depends on several other LAPACK routines:
- DLARFG (generate Householder reflector)
- DGEMV (matrix-vector multiplication)
- DGER (rank-1 update)
- DLARF (apply Householder reflector)
- DAXPY, DCOPY (BLAS level 1)
- DGEMM (BLAS level 3, for block algorithm)

#### Header Files (include/)

1. **include/stdlib/lapack/base/dgehrd.h**: C function declarations

### Phase 5: Benchmarks

**Effort**: Medium  
**Artifacts**: benchmark/*.js

- Benchmark against different matrix sizes
- Compare native vs JavaScript implementations
- Test performance with different ilo/ihi values
- ~2-3 benchmark files, ~100-200 lines total

### Phase 6: Tests

**Effort**: High  
**Artifacts**: test/*.js

Requirements:
- **100% code coverage** (enforced)
- Test all parameter combinations
- Edge cases:
  - Empty matrices
  - Single element
  - Various ilo/ihi combinations
  - Different data layouts (row-major, column-major)
  - Invalid inputs (negative dimensions, out-of-bounds indices)
- Numerical accuracy tests (compare against known results)
- Test both JavaScript and native implementations
- ndarray interface tests

Estimated: ~400-600 lines of test code

### Phase 7: REPL Documentation

**Effort**: Low  
**Artifacts**: docs/repl.txt

- Concise help text for REPL usage
- ~20-40 lines

### Phase 8: TypeScript Declarations

**Effort**: Medium  
**Artifacts**: docs/types/index.d.ts, docs/types/test.ts

- Define TypeScript interfaces and types
- Create type tests
- ~100-150 lines total

### Phase 9: Build Configuration

**Effort**: Medium  
**Artifacts**: binding.gyp, include.gypi, manifest.json

- Configure Node.js native add-on build
- Specify compiler flags, include paths
- Link against LAPACK library or include source
- ~50-100 lines configuration

## Dependencies and Prerequisites

### External Dependencies

1. **LAPACK library** (if using external implementation):
   - Need LAPACK/BLAS installed on build system
   - Increases complexity for users building from source
   
2. **Reference implementation** (if porting):
   - Fortran DGEHRD from Netlib LAPACK (~500 lines)
   - Requires porting Fortran to C
   - Need to port dependent routines (DLARFG, DLARF, etc.)

### Internal Dependencies

Based on existing stdlib LAPACK packages, xGEHRD would need:

- `@stdlib/array/float64`
- `@stdlib/lapack/base/shared` (existing shared utilities)
- `@stdlib/assert/*` (for validation)
- `@stdlib/error/*` (for error handling)
- `@stdlib/string/format`

If implementing from scratch, may need:
- `@stdlib/lapack/base/dlarfg` (generate Householder reflector) - **NOT YET IMPLEMENTED**
- `@stdlib/lapack/base/dlarf1f` (apply Householder reflector) - **✓ EXISTS**
- `@stdlib/blas/base/dgemv` (matrix-vector product) - **✓ EXISTS**
- `@stdlib/blas/base/dger` (rank-1 update) - **✓ EXISTS**
- `@stdlib/blas/base/daxpy` (vector addition) - **✓ EXISTS**
- `@stdlib/blas/base/dcopy` (vector copy) - **✓ EXISTS**
- `@stdlib/blas/base/dscal` (vector scaling) - **✓ EXISTS**
- `@stdlib/blas/base/dgemm` (matrix-matrix product) - **✓ EXISTS**

### Dependency Status Assessment

**Good news**: Most required dependencies already exist in stdlib!

- ✅ **BLAS Level 1**: All required routines exist (DAXPY, DCOPY, DSCAL)
- ✅ **BLAS Level 2**: DGEMV and DGER exist
- ✅ **BLAS Level 3**: DGEMM exists for block algorithm
- ✅ **DLARF1F**: Exists for applying Householder reflectors
- ⚠️ **DLARFG**: Not found - this is a **critical missing dependency**

**DLARFG Status**: This routine generates Householder reflectors and is essential for GEHRD. It must be implemented before GEHRD. However, DLARFG is a relatively simple routine (~100-150 lines) compared to GEHRD itself.

**Impact**: The missing DLARFG reduces the overall implementation complexity since most other dependencies exist. DLARFG should be implemented as a prerequisite package first.

## Complexity Assessment

### Lines of Code Estimate

| Component | Estimated LOC | Complexity |
|-----------|---------------|------------|
| README.md | 200-300 | Medium |
| lib/*.js | 600-1000 | High |
| src/*.c | 400-700 | High |
| include/*.h | 50-100 | Low |
| test/*.js | 400-600 | High |
| benchmark/*.js | 100-200 | Medium |
| examples/*.js | 100-150 | Low |
| docs/types/* | 100-150 | Medium |
| Other (package.json, gyp, etc.) | 100-150 | Low |
| **Total** | **2050-3350** | **High** |

### Time Estimate

For an experienced developer familiar with:
- LAPACK algorithms
- stdlib conventions
- C/JavaScript/Node.js native add-ons

**Estimated time**: 40-80 hours (1-2 weeks full-time)

Breakdown:
- Research & planning: 4-8 hours
- README & design: 4-6 hours  
- Core implementation (JS + C): 15-25 hours
- Tests (to 100% coverage): 10-20 hours
- Benchmarks: 3-5 hours
- Documentation & types: 3-5 hours
- Build configuration & debugging: 5-10 hours

For a developer new to LAPACK or stdlib:
**Estimated time**: 80-120+ hours (2-3 weeks full-time)

## Implementation Strategies

### Option 1: Link Against External LAPACK

**Pros**:
- Leverages highly optimized, well-tested code
- Less code to maintain
- Faster to implement

**Cons**:
- External dependency complicates build
- Users need LAPACK installed
- Less portable
- Inconsistent with some stdlib packages

### Option 2: Port Reference Implementation

**Pros**:
- Self-contained (no external dependencies)
- Full control over implementation
- Consistent with stdlib philosophy

**Cons**:
- Significant effort to port Fortran to C
- Need to port multiple dependent routines
- Requires thorough testing to ensure correctness
- More code to maintain

### Option 3: Implement from Scratch

**Pros**:
- Optimized for stdlib's needs
- Educational value
- Maximum control

**Cons**:
- Highest effort
- Risk of bugs in numerical algorithm
- Need deep understanding of Householder reduction
- Extensive testing required

## Recommended Approach

Based on existing stdlib LAPACK packages (e.g., dlacpy), the project appears to favor **Option 2** (self-contained implementations). However, GEHRD is significantly more complex than existing packages like DLACPY or DLASET.

**Recommended phased approach**:

1. **Phase 0**: Implement DLARFG (prerequisite, ~1-2 days)
   - Generates Householder reflectors
   - ~150-200 lines of code
   - Simpler than GEHRD, good warm-up
2. **Phase 1**: Start with double-precision (DGEHRD)
   - Most commonly used variant
   - ~1-2 weeks for complete implementation
3. **Phase 2**: Extend to single precision (SGEHRD)
   - Straightforward port from DGEHRD
   - ~2-3 days
4. **Phase 3**: Implement complex variants (CGEHRD, ZGEHRD)
   - Requires complex number handling
   - ~1 week for both

**Revised Time Estimate** (with existing dependencies):

Given that most BLAS and DLARF1F exist:
- DLARFG implementation: 16-24 hours (2-3 days)
- DGEHRD implementation: 50-70 hours (1-2 weeks)
- **Total for DLARFG + DGEHRD: 66-94 hours (1.5-2.5 weeks)**

## Challenges and Risks

### Technical Challenges

1. **Algorithm Complexity**: GEHRD is more complex than simple matrix operations
2. **Numerical Stability**: Householder transformations require careful implementation
3. **Dependencies**: May need to implement several prerequisite LAPACK routines
4. **Testing**: Difficult to generate comprehensive test cases for numerical accuracy
5. **Performance**: Pure JavaScript implementation may be slow for large matrices

### Development Risks

1. **Time Investment**: Larger than typical stdlib package
2. **Testing Coverage**: Achieving 100% coverage with numerical accuracy is challenging
3. **Maintenance**: Complex numerical code requires ongoing maintenance
4. **Build System**: Native add-on configuration can be tricky across platforms

## Conclusion

### Feasibility: **FEASIBLE** and **MORE TRACTABLE THAN INITIALLY ASSESSED**

Implementing xGEHRD is feasible within stdlib's framework. The discovery that most dependencies already exist makes this more tractable than initially estimated:

**Positive Factors:**
- ✅ All required BLAS routines exist (Level 1, 2, and 3)
- ✅ DLARF1F exists for applying Householder reflectors
- ✅ Existing LAPACK packages provide clear implementation patterns
- ✅ stdlib has comprehensive development documentation

**Remaining Challenges:**
- ⚠️ DLARFG must be implemented first (prerequisite)
- **Algorithm complexity**: GEHRD is still more complex than existing LAPACK packages
- **Testing requirements**: 100% coverage with numerical accuracy validation
- **Time investment**: ~2-3 weeks total for DLARFG + DGEHRD

**Complexity Reassessment:**

Original estimate: High complexity, 80-120+ hours
**Revised estimate**: Medium-High complexity, 66-94 hours
- DLARFG: 16-24 hours (prerequisite)
- DGEHRD: 50-70 hours (main implementation)

### Recommendations

1. **Implement DLARFG first**: This is the only missing critical dependency
   - Simpler routine (~150-200 LOC)
   - Good introduction to LAPACK development in stdlib
   - Can be done as a separate PR
2. **Start with DGEHRD only**: Don't attempt all four variants initially
3. **Thorough research**: Study the reference LAPACK implementation carefully
4. **Incremental development**: Follow the recommended development order strictly
5. **Extensive testing**: Allocate significant time for test development
6. **Performance validation**: Benchmark against reference implementations
7. **Consider two PRs**: 
   - PR #1: Implement DLARFG (and SLARFG, CLARFG, ZLARFG)
   - PR #2: Implement DGEHRD (leveraging DLARFG)

### Developer Skills Required

- Strong understanding of linear algebra
- Experience with LAPACK/BLAS
- Proficiency in C and JavaScript
- Familiarity with Node.js native add-ons
- Knowledge of numerical computing best practices
- Experience with stdlib conventions

### Next Steps if Proceeding

1. ✅ **Confirmed**: Most BLAS dependencies exist (DGEMV, DGER, DAXPY, DCOPY, DSCAL, DGEMM)
2. ✅ **Confirmed**: DLARF1F exists for applying Householder reflectors
3. ⚠️ **Missing**: DLARFG must be implemented first
4. **Implement DLARFG** as prerequisite:
   - Review LAPACK DLARFG reference implementation
   - Create @stdlib/lapack/base/dlarfg package
   - Follow stdlib package development guide
   - Estimated time: 2-3 days
5. **Implement DGEHRD** after DLARFG is complete:
   - Review LAPACK DGEHRD reference implementation
   - Create @stdlib/lapack/base/dgehrd package
   - Begin with README-first development
   - Estimated time: 1-2 weeks

---

**Document Status**: Complete  
**Assessment Date**: 2025-11-22  
**Assessor**: GitHub Copilot  
**Review Recommended**: Yes - by maintainer with LAPACK expertise
