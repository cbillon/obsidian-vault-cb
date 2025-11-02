---
tags:
  - go
  - quality
link: https://www.grootan.com/blogs/ensuring-code-quality-in-go-a-comprehensive-guide-to-standard-checks/
---
## Key Checks in the Script

The provided steps ensure Go checks to enforce best practices.

1. **Code Formatting** (`go fmt`)
2. **Static Code Analysis** (`go vet`)
3. **Import Formatting** (`goimports`)
4. **Linting** (`golangci-lint`)
5. **Swagger Documentation** (`swag init`)
6. **Dependency Management** (`go mod tidy`)
7. **Project Build** (`go build`)
8. **Security Checks** (`gosec`)
9. **Secret Leak Detection** (`gitleaks`)
10. **Unit Testing** (`go test`)
11. **Test Coverage Analysis** (`go test -cover`)

## Breakdown of Each Step with Examples

### 1. Code Formatting (`go fmt`)

- Ensures a consistent code style by formatting the Go source files.
- **Command**: `go fmt ./...`

**Example (Before Formatting):**

```go
package main
import "fmt"func main() {
fmt.Println("Hello, World!")
}
```

**After Running `go fmt`:**

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### 2. Static Code Analysis (`go vet`)

- Detects issues like unreachable code, incorrect struct tags, and suspicious constructs.
- **Command**: `go vet ./...`

**Example:**

```go
package main

import "fmt"

func main() {
    var unusedVar int  // go vet will flag this as an unused variable
    fmt.Println("Hello, world!")
}
```

**Output from `go vet`:**

```
main.go:5:6: unused variable 'unusedVar'
```

### 3. Import Formatting (`goimports`)

- Formats and organizes import statements.
- **Command**: `goimports -w .`

**Example (Before Formatting):**

```go
import ("fmt" "os")
```

**After Running `goimports`:**

```go
import (
    "fmt"
    "os"
)
```

### 4. Linting (`golangci-lint`)

- Performs static analysis, style enforcement, and best-practice checks.
- **Command**: `golangci-lint run`

**Example:**

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello") // golangci-lint will warn if error handling is missing
}
```

### 5. Swagger Documentation (`swag init`)

- Generates and updates API documentation.
- **Command**: `swag init`

**Example:**

```go
// @Summary Get user by ID
// @Description Get details of a user
// @Param id path int true "User ID"
// @Success 200 {object} User
// @Router /users/{id} [get]
func GetUserByID(c *gin.Context) {
    // Handler logic
}
```

### 6. Dependency Management (`go mod tidy`)

- Ensures that only necessary dependencies are listed in `go.mod`.
- **Command**: `go mod tidy && go list -m all`

### 7. Project Build (`go build`)

- Compiles the project to validate build correctness.
- **Command**: `go build ./...`

### 8. Security Checks (`gosec`)

- Scans code for security vulnerabilities.
- **Command**: `gosec ./...`

**Example:**

```go
package main

func main() {
    password := "mypassword" // gosec will flag this as a potential security risk
}
```

### 9. Secret Leak Detection (`gitleaks`)

- Checks for exposed secrets in the repository.
- **Command**: `gitleaks detect . --verbose`

**Example:**

```go
package main

func main() {
    apiKey := "sk_test_123456789" // gitleaks will detect this as a sensitive key
}
```

### 10. Unit Testing (`go test`)

- Runs unit tests to validate functionality.
- **Command**: `go test ./...`

**Example:**

```go
package math

import "testing"

func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, but got %d", result)
    }
}
```

### 11. Test Coverage Analysis (`go test -cover`)

- Measures code coverage.
- **Command**: `go test -cover ./...`

**Example:**

```sh
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## Complete Automation Script

To automate these checks, use the following script:

```sh
#!/bin/bash

# ============================ PARAMETERS ============================
# Set default flags to execute all steps
RUN_FMT=true
RUN_VET=true
RUN_IMPORTS=true
RUN_LINT=true
RUN_SWAG=true
RUN_TIDY=true
RUN_BUILD=true
RUN_SECURITY=true
RUN_GIT_LEAKS=true
RUN_TEST=true

# Parse flags
while [[ "$#" -gt 0 ]]; do
    case $1 in
        --skip-fmt) RUN_FMT=false ;;
        --skip-vet) RUN_VET=false ;;
        --skip-imports) RUN_IMPORTS=false ;;
        --skip-lint) RUN_LINT=false ;;
        --skip-swag) RUN_SWAG=false ;;
        --skip-tidy) RUN_TIDY=false ;;
        --skip-build) RUN_BUILD=false ;;
        --skip-security) RUN_SECURITY=false ;;
        --skip-git-leak) RUN_GIT_LEAKS=false ;;
        --skip-test) RUN_TEST=false ;;
        *) echo "Unknown parameter: $1" ;;
    esac
    shift
done

# ============================ STATUS TRACKING ============================
STEPS=("go fmt" "swag init" "go vet" "Tidying modules" "goimports" "golangci-lint" "Building project" "Security checks (gosec)" "Git leaks check" "Unit tests" "Test coverage")
STATUSES=() # Parallel array for statuses
PASSED_COUNT=0
FAILED_COUNT=0

# ============================ FUNCTIONS ============================
log_banner() {
    echo ""
    echo "==================================================================="
    echo "  $1  :  $2" 
    echo "==================================================================="
    echo ""
}

run_step() {
    local step_name="$1"
    local command="$2"
    local description="$3"

    log_banner "Executing: $step_name" "$command" "$description"
    eval "$command"
    if [[ $? -eq 0 ]]; then
        echo "✔️  $step_name completed successfully."
        STATUSES+=("✔️ > $step_name: PASSED")
        PASSED_COUNT=$((PASSED_COUNT + 1))
    else
        echo "❌  $step_name failed."
        STATUSES+=("❌> $step_name: FAILED > Run: '$command'"  )
        FAILED_COUNT=$((FAILED_COUNT + 1))
    fi
}

print_status_board() {
    echo ""
    echo "==================================================================="
    echo "                     FINAL STATUS BOARD                            "
    echo "==================================================================="
    for status in "${STATUSES[@]}"; do
        echo "$status"
    done
    echo "==================================================================="
    echo "                     SUMMARY                                       "
    echo "==================================================================="
    echo "✔️  Passed: $PASSED_COUNT"
    echo "❌  Failed: $FAILED_COUNT"
    echo "==================================================================="
    echo ""
}

# ============================ EXECUTION ============================
if $RUN_FMT; then
    run_step "Formatting" "go fmt ./..." "Code Formatting and Style"
else
    STATUSES+=("go fmt: SKIPPED")
fi

if $RUN_SWAG; then
    run_step "Swag Init" "swag init --dir cmd,internal,pkg --output ./docs --parseDependency --parseFuncBody --parseVendor --parseDependencyLevel 3 --parseInternal" "Updates swagger doc"
else
    STATUSES+=("swag init: SKIPPED")
fi

if $RUN_VET; then
    run_step "Code Analysis" "go vet ./..." "Static Code Analysis: Detects issues like unreachable code, incorrect struct tags, and more."
else
    STATUSES+=("go vet: SKIPPED")
fi

if $RUN_TIDY; then
    run_step "Dependency Management" "go mod tidy && go list -m all" "Dependency Management : Clean up and Ensures none are missing"
else
    STATUSES+=("Tidying modules: SKIPPED")
fi

if $RUN_IMPORTS; then
    run_step "Import formatting" "goimports -w ." "Check import Formatting and Ordering"
else
    STATUSES+=("goimports: SKIPPED")
fi

if $RUN_LINT; then
    run_step "LINT" "golangci-lint run" "Code Linting: Ensures error handling , standards, bug findings, suggestions"
else
    STATUSES+=("golangci-lint: SKIPPED")
fi

if $RUN_BUILD; then
    run_step "Build" "go build ./..." "Compiles the project to check for build errors."
else
    STATUSES+=("Building project: SKIPPED")
fi

if $RUN_SECURITY; then
    run_step "Security Check" "gosec ./..." "Security Checks : vulnerabilities"
else
    STATUSES+=("Security checks (gosec): SKIPPED")
fi

if $RUN_GIT_LEAKS; then
    run_step "Git Leaks Check" "gitleaks detect . --verbose" "Check for secrets exposing"
else
    STATUSES+=("Git leaks check: SKIPPED")
fi

if $RUN_TEST; then
    run_step "Unit tests" "go test ./..." "Unit test Executions" 
    run_step "Test coverage" "go test -cover ./..." "Code coverage validations"
else
    STATUSES+=("Unit Testing: SKIPPED")
    STATUSES+=("Test Coverage: SKIPPED")
fi

log_banner "ALL CHECKS COMPLETED!"
print_status_board

# ============================ END OF SCRIPT ============================
```

# Conclusion

Automating these checks ensures that Go applications maintain high-quality standards. By integrating these steps into your CI/CD pipeline, you can enforce best practices, prevent regressions, and improve team collaboration.