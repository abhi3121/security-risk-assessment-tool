# Framework reference — build, test, conventions, failing-test skeletons

Load only the rows that match your detected toolchain. Do **not** copy this
whole file into the agent's working context.

Used by:
- `capabilities/init.md` step "detect host repo context" — to know which
  detection signals to look for in the host repo.
- `capabilities/tests.md` step "confirm test framework" and "generate failing
  skeletons" — to pick file locations and the right skeleton snippet.

---

## 1. Build-system detection signals

| Build system | Signals (files / patterns) |
|--------------|---------------------------|
| CMake        | `CMakeLists.txt`, `CMakePresets.json` |
| Make         | `Makefile`, `GNUmakefile` |
| Meson        | `meson.build` |
| npm / pnpm / yarn | `package.json` (+ lockfile) |
| Python (PEP 517) | `pyproject.toml`, `setup.cfg`, `setup.py`, `requirements*.txt` |
| Cargo        | `Cargo.toml` |
| Go           | `go.mod` |
| Maven        | `pom.xml` |
| Gradle       | `build.gradle`, `build.gradle.kts`, `settings.gradle(.kts)` |
| .NET / MSBuild | `*.sln`, `*.csproj`, `*.fsproj`, `*.vbproj`, `Directory.Build.props` |
| Bazel        | `WORKSPACE`, `WORKSPACE.bazel`, `BUILD`, `BUILD.bazel`, `MODULE.bazel` |
| Docker       | `Dockerfile`, `compose.yaml`, `docker-compose.yml` |
| Composer (PHP) | `composer.json` |
| Bundler (Ruby) | `Gemfile`, `Gemfile.lock` |

If multiple are present, list them all; ask the user which is authoritative
for the target subsystem.

---

## 2. Test framework detection signals

| Language family | Candidate frameworks (signals) |
|-----------------|-------------------------------|
| C / C++ | gtest + ctest (`gtest/` headers, `add_test()` in CMake), Catch2 (`catch.hpp`, `catch_amalgamated.hpp`), Unity (`unity.h`), cmocka (`cmocka.h`), Criterion (`criterion/criterion.h`) |
| Python | pytest (`pytest.ini`, `pyproject.toml [tool.pytest]`, `conftest.py`), unittest (stdlib, `test_*.py` with `unittest.TestCase`), nose2 (`nose2.cfg`) |
| JS / TS | jest (`jest.config.*`), vitest (`vitest.config.*`), mocha (`.mocharc.*`), Playwright (`playwright.config.*`), Cypress (`cypress.config.*`), Storybook test-runner (`@storybook/test-runner`), `@testing-library/*`, Vue Test Utils, Angular TestBed (Karma/Jasmine) |
| Java / Kotlin | JUnit 5 (`junit-jupiter`), JUnit 4 (`junit:junit`), TestNG (`org.testng`), Spock (Groovy) |
| Go | built-in `go test` + `*_test.go`; testify (`github.com/stretchr/testify`); ginkgo |
| Rust | built-in `#[test]` + `tests/` dir; proptest |
| C# / .NET | xUnit (`xunit` package), NUnit (`NUnit` package), MSTest (`MSTest.TestFramework`) |
| Ruby | rspec (`spec/` dir, `.rspec`), minitest |
| PHP | PHPUnit (`phpunit.xml*`), Pest (`pest.config.php`, `Pest.php`) |
| Shell | bats (`*.bats`, `bats-core`) |
| Mobile | Detox (React Native: `.detoxrc.*`), Espresso (Android: `androidx.test.espresso`), XCUITest (iOS: `*UITests/`) |
| UI / Web e2e | Playwright, Cypress, Selenium (`selenium-*`), WebdriverIO (`wdio.conf.*`) |

If detection is ambiguous, list candidates and ask the user to confirm.

---

## 3. Conventional test file locations & naming

| Framework | Typical location | File-name pattern |
|-----------|-----------------|-------------------|
| gtest + ctest    | `<module>/tests/` or `tests/`               | `ut_<feature>.cc`, `it_<feature>.cc` |
| Catch2           | `tests/`                                     | `test_<feature>.cpp` |
| pytest           | `tests/`                                     | `test_<feature>.py` |
| unittest (py)    | `tests/`                                     | `test_<feature>.py` (class derives `unittest.TestCase`) |
| Jest / Vitest    | `src/<area>/__tests__/` or `tests/`          | `<feature>.test.ts` |
| Mocha            | `test/`                                      | `<feature>.spec.js` |
| Playwright       | `tests/e2e/` or `e2e/`                       | `<feature>.spec.ts` |
| Cypress          | `cypress/e2e/`                               | `<feature>.cy.ts` |
| JUnit 5          | `src/test/java/<package>/`                   | `<Feature>Test.java` |
| TestNG           | `src/test/java/<package>/`                   | `<Feature>Test.java` |
| Go test          | `<pkg>/`                                     | `<feature>_test.go` |
| Rust (built-in)  | `tests/` (integration) or `#[cfg(test)] mod` | `<feature>.rs` |
| xUnit / NUnit    | `<Project>.Tests/`                           | `<Feature>Tests.cs` |
| RSpec            | `spec/`                                      | `<feature>_spec.rb` |
| PHPUnit          | `tests/`                                     | `<Feature>Test.php` |
| Pest             | `tests/Feature/` or `tests/Unit/`            | `<Feature>Test.php` |
| bats             | `tests/`                                     | `<feature>.bats` |

Honour any existing project convention you find — if the host repo deviates,
ask the user before introducing a new convention.

---

## 4. Failing-test skeleton patterns (RED state)

Every skeleton must **fail by default** with a message that contains the `TC-n`
id verbatim so `grep TC-` works across the tree.

| Framework | Skeleton |
|-----------|----------|
| gtest      | `TEST(Feature, TC_1_keygen_happy) { FAIL() << "TC-1 not implemented"; }` |
| Catch2     | `TEST_CASE("TC-1 keygen happy", "[feature]") { FAIL("TC-1 not implemented"); }` |
| pytest     | `def test_tc_1_keygen_happy(): pytest.fail("TC-1 not implemented")` |
| unittest   | `def test_tc_1_keygen_happy(self): self.fail("TC-1 not implemented")` |
| Jest       | `test("TC-1 keygen happy", () => { throw new Error("TC-1 not implemented"); });` |
| Vitest     | `test("TC-1 keygen happy", () => { throw new Error("TC-1 not implemented"); });` |
| Mocha      | `it("TC-1 keygen happy", () => { throw new Error("TC-1 not implemented"); });` |
| JUnit 5    | `@Test void tc_1_keygen_happy() { fail("TC-1 not implemented"); }` |
| TestNG     | `@Test public void tc_1_keygen_happy() { Assert.fail("TC-1 not implemented"); }` |
| Go         | `func TestTC1KeygenHappy(t *testing.T) { t.Fatal("TC-1 not implemented") }` |
| Rust       | `#[test] fn tc_1_keygen_happy() { panic!("TC-1 not implemented"); }` |
| xUnit      | `[Fact] public void TC_1_KeygenHappy() { Assert.Fail("TC-1 not implemented"); }` |
| NUnit      | `[Test] public void TC_1_KeygenHappy() { Assert.Fail("TC-1 not implemented"); }` |
| MSTest     | `[TestMethod] public void TC_1_KeygenHappy() { Assert.Fail("TC-1 not implemented"); }` |
| RSpec      | `it "TC-1 keygen happy" do; fail "TC-1 not implemented"; end` |
| PHPUnit    | `public function testTC1KeygenHappy(): void { $this->fail("TC-1 not implemented"); }` |
| Pest       | `it('TC-1 keygen happy', function () { $this->fail('TC-1 not implemented'); });` |
| bats       | `@test "TC-1 keygen happy" { fail "TC-1 not implemented"; }` |
| Playwright | `test("TC-1 happy path", async ({ page }) => { throw new Error("TC-1 not implemented"); });` |
| Cypress    | `it("TC-1 happy path", () => { throw new Error("TC-1 not implemented"); });` |

For any framework not listed above, follow the same pattern: use the
framework's idiomatic "fail" / "skip-with-failure" primitive, and include the
literal `TC-n` id in the test name.

---

## 5. Wiring new test files into the build

| Build system | Wiring step |
|--------------|-------------|
| CMake (gtest/ctest) | Add the file to the relevant `add_executable` / `target_sources`, ensure `add_test(NAME ... COMMAND ...)` is registered. Re-run cmake configure if presets are used. |
| Make (manual)       | Add the file to `TESTS` / `OBJS` rule in the Makefile. |
| Meson               | Add to `test_sources` array and ensure `test()` call exists. |
| Jest / Vitest       | Default glob usually picks it up; verify the file matches `testMatch` / `include`. |
| pytest              | Default discovery picks up `test_*.py` under `tests/`. |
| Go                  | Implicit (`go test ./...`). |
| Cargo               | Implicit for `tests/<x>.rs` or `#[cfg(test)]`. |
| Maven / Gradle      | Default discovery under `src/test/java/`. |
| .NET                | Default discovery for `*.Tests` projects. |
| Bazel               | Add a `cc_test` / `py_test` / equivalent rule in the local `BUILD` file. |

If the host repo uses a non-default convention, follow that convention rather
than the table above.
