# Java 8 → Java 17 Upgrade Report

**Project:** NextGen Connect Integration Engine (Mirth Connect)  
**Build System:** Apache Ant  
**Previous Java Version:** 8  
**Target Java Version:** 17 (LTS)  
**Date:** 2026-03-05  

---

## 1. Build Configuration Changes

All 15 Ant build files were updated to target Java 17:

### Compilation Target Updates (`release="17"`)

| Module | Build File | Compile Tasks Updated |
|--------|------------|----------------------|
| core-models | `core-models/build.xml` | compile, test-compile |
| core-util | `core-util/build.xml` | compile, test-compile |
| core-client-base | `core-client-base/build.xml` | compile, test-compile |
| core-client-api | `core-client-api/build.xml` | compile, test-compile |
| core-client | `core-client/build.xml` | compile, test-compile |
| core-server-plugins | `core-server-plugins/build.xml` | compile, test-compile |
| core-ui | `core-ui/build.xml` | compile, test-compile |
| core-client-plugins | `core-client-plugins/build.xml` | compile, test-compile |
| donkey | `donkey/build.xml` | compile, test-compile |
| server | `server/build.xml` | compile, test-compile |
| command | `command/build.xml` | compile, test-compile |
| client | `client/ant-build.xml` | compile, test-compile |
| manager | `manager/ant-build.xml` | compile |
| webadmin | `webadmin/build.xml` | compile (+ jspc sourceVM/targetVM) |
| generator | `generator/build.xml` | compile-generator, compile-vocab |

### Key Build Changes

- **`release="17"` attribute**: Added to all `<javac>` tasks, replacing the implicit Java 8 default. The `release` attribute is preferred over separate `source`/`target` attributes as it also sets the boot classpath correctly.
- **webadmin JSP compiler**: Updated `compilerSourceVM` and `compilerTargetVM` from `"1.6"` to `"17"`.
- **Javadoc**: Updated API link from `https://docs.oracle.com/javase/8/docs/api/` to `https://docs.oracle.com/en/java/javase/17/docs/api/` and enabled `-html5` output.

---

## 2. Java Module System (JPMS) Changes

Java 9+ introduced the module system which restricts reflective access to internal JDK APIs. The following `--add-opens` JVM arguments were activated in all test-run targets:

```
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.text=ALL-UNNAMED
--add-opens=java.desktop/java.awt=ALL-UNNAMED
--add-opens=java.desktop/java.awt.font=ALL-UNNAMED
--add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED
```

These were already present as comments in many build files (prepared for Java 9+ migration). They have been uncommented and standardized across all modules with proper `--` (double-dash) prefix format.

### Removed Obsolete Compiler Args

The following commented-out compiler arguments in `server/build.xml` were removed as they reference Java EE modules that were fully removed in Java 11:

```xml
<!-- Removed - modules no longer exist in JDK -->
<compilerarg value="--add-modules" />
<compilerarg value="java.sql.rowset,java.xml.ws" />
<compilerarg value="--add-exports" />
<compilerarg value="java.sql.rowset/com.sun.rowset=ALL-UNNAMED" />
```

---

## 3. Deprecated/Removed API Analysis

### Java EE Modules (removed in Java 11)

| API | Status | Mitigation |
|-----|--------|------------|
| `javax.xml.bind` (JAXB) | Removed from JDK | **Already mitigated** — standalone `jaxb-api-2.4.0` and `jaxb-runtime-2.4.0` JARs in `server/lib/javax/jaxb/` |
| `javax.xml.ws` (JAX-WS) | Removed from JDK | **Already mitigated** — standalone `jaxws-api-2.3.0` and `jaxws-rt-2.3.0.2` JARs in `server/lib/javax/jaxws/` |
| `javax.annotation` | Removed from JDK | **Already mitigated** — standalone `javax.annotation-api-1.3.1.jar` in `server/lib/javax/jaxws/` |
| `javax.activation` | Removed from JDK | **Already mitigated** — standalone `javax.activation-1.2.0.jar` in `server/lib/javax/jaxb/` |

Since the project already included standalone JAR replacements for all removed Java EE modules, **no source code import changes were required**.

### Other API Observations

| API | Status in Java 17 | Action |
|-----|-------------------|--------|
| `com.sun.net.httpserver` | Still available (jdk.httpserver module) | No change needed |
| `SecurityManager` | Deprecated for removal | Used with null-guard in `MirthJavaScriptThreadFactory.java` — safe, no runtime impact |
| `java.util.logging.LogManager` | Still available | No change needed |

---

## 4. Dependency Compatibility

All existing library JARs in `server/lib/` are compatible with Java 17:

- **JAXB 2.4.0** — Provides `javax.xml.bind.*` as standalone library ✓
- **JAX-WS 2.3.0** — Provides `javax.xml.ws.*` as standalone library ✓  
- **Rhino JavaScript Engine** — Bundled in `core-util/`, not dependent on Nashorn ✓
- **Log4j2** — Used for logging, compatible with Java 17 ✓
- **JUnit 4** — Compatible with Java 17 (with `--add-opens` for reflective access) ✓
- **Jacoco** — Code coverage agent, compatible with Java 17 ✓

---

## 5. Files Modified

15 build configuration files across all modules:

1. `client/ant-build.xml`
2. `command/build.xml`
3. `core-client-api/build.xml`
4. `core-client-base/build.xml`
5. `core-client-plugins/build.xml`
6. `core-client/build.xml`
7. `core-models/build.xml`
8. `core-server-plugins/build.xml`
9. `core-ui/build.xml`
10. `core-util/build.xml`
11. `donkey/build.xml`
12. `generator/build.xml`
13. `manager/ant-build.xml`
14. `server/build.xml`
15. `webadmin/build.xml`

**No Java source files were modified** — the existing standalone JAR dependencies already provide all APIs removed from the JDK.

---

## 6. Migration Notes

### Runtime Considerations

When deploying the application with Java 17, the following JVM arguments should be included in the startup scripts:

```
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.text=ALL-UNNAMED
--add-opens=java.desktop/java.awt=ALL-UNNAMED
--add-opens=java.desktop/java.awt.font=ALL-UNNAMED
--add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED
```

### Future Considerations

- **SecurityManager**: Deprecated for removal (JEP 411). The usage in `MirthJavaScriptThreadFactory` is guarded and will gracefully degrade. Consider removing the SecurityManager code path in a future update.
- **Jakarta namespace migration**: The project currently uses `javax.*` namespace via standalone JAXB 2.4 / JAX-WS 2.3 JARs. A future upgrade could migrate to Jakarta EE 9+ (`jakarta.*` namespace) with JAXB 3.0+ / JAX-WS 3.0+.
