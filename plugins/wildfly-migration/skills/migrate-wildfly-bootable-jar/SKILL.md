---
name: migrate-wildfly-bootable-jar
description: Migrate from WildFly JAR Maven Plugin to WildFly Maven Plugin for bootable JAR support
args: "[pom-file-path]"
---

You are helping migrate a Maven project from the WildFly JAR Maven Plugin to the WildFly Maven Plugin.

## Steps to follow:

1. **Find the pom.xml**: 
   - If the user provided a path as an argument, use that
   - Otherwise, look for `pom.xml` in the current directory

2. **Read and analyze the pom.xml**:
   - Check if it uses `wildfly-jar-maven-plugin`
   - If not found, inform the user and exit
   - If found, identify all configuration elements that need migration

3. **Show the user what will change**:
   - List all the changes that will be made
   - Ask for confirmation before proceeding

4. **Perform the migration** (after user confirms):

   **Plugin changes**:
   - Replace `<artifactId>wildfly-jar-maven-plugin</artifactId>` with `<artifactId>wildfly-maven-plugin</artifactId>`
   - Update `<version>` to `6.0.0.Final`
   - Add `<bootable-jar>true</bootable-jar>` in the configuration section

   **Feature-pack location migration**:
   - If using `<feature-pack-location>`, convert to `<feature-packs><feature-pack><location>` structure:
     ```xml
     <!-- OLD -->
     <feature-pack-location>org.wildfly:wildfly-galleon-pack:40.0.0.Final</feature-pack-location>
     
     <!-- NEW -->
     <feature-packs>
         <feature-pack>
             <location>org.wildfly:wildfly-galleon-pack:40.0.0.Final</location>
         </feature-pack>
     </feature-packs>
     ```

   **Configuration element renames**:
   - `<plugin-options>` → `<galleon-options>`
   - `<cli-sessions>` → `<packaging-scripts>`
   - `<cli-session>` → `<packaging-script>`
   - `<script-files>` → `<scripts>`
   - `<output-file-name>` → `<bootable-jar-name>`
   - `<offline>` → `<offline-provisioning>`
   - `<log-time>` → `<log-provisioning-time>`
   - `<record-state>` → `<record-provisioning-state>`
   - `<hollow-jar>` → `<skip-deployment>` (if hollow-jar is true, set skip-deployment to true)
   - `<context-root>` → Remove this element; to deploy in root context, use `<name>ROOT.war</name>`

   **Cloud migration**:
   - If `<cloud/>` element exists, remove it
   - Add to feature-packs list:
     ```xml
     <feature-pack>
         <location>org.wildfly.cloud:wildfly-cloud-galleon-pack:9.2.3.Final</location>
     </feature-pack>
     ```

5. **Verify the changes**:
   - Read the updated pom.xml
   - Confirm all changes were applied correctly

6. **Provide next steps**:
   - Remind user to update build scripts/CI/CD with new goals:
     - `wildfly-jar:package` → `wildfly:package`
     - `wildfly-jar:start` → `wildfly:start-jar`
     - `wildfly-jar:run` → `wildfly:run`
     - `wildfly-jar:dev` → `wildfly:dev`
     - `wildfly-jar:shutdown` → `wildfly:shutdown`
   - Suggest testing with `mvn clean wildfly:package`
   - Point to the full migration guide at `https://github.com/jfdenise/wildfly.org/blob/migrate_bootable_jar/content/guides/wildfly-jar-migration.adoc`

## Important notes:

- **`<bootable-jar>true</bootable-jar>` is a boolean flag**, not a wrapper element - this is REQUIRED to enable bootable JAR mode
- All configuration elements remain at the same level in the structure
- The most significant changes are:
  - `<feature-pack-location>` must be converted to nested `<feature-packs><feature-pack><location>` structure
  - Cloud element removal - must use feature-pack instead
  - Context root deployment now uses `<name>ROOT.war</name>` instead of `<context-root>true</context-root>`
- Default bootable JAR filename changed from `${project.build.finalName}-bootable.jar` to `${project.artifactId}-bootable.jar`
- WildFly version in examples: 40.0.0.Final
- Cloud feature-pack version: 9.2.3.Final
- WildFly Maven Plugin version: 6.0.0.Final

## Common migration issues to watch for:

1. **Missing bootable-jar flag**: Without `<bootable-jar>true</bootable-jar>`, the plugin builds a regular provisioned server instead of a bootable JAR
2. **Wrong goal names**: Update all `wildfly-jar:*` goals to `wildfly:*` in scripts and CI/CD
3. **Cloud element not recognized**: The `<cloud/>` element no longer exists - must use the cloud feature-pack
4. **Bootable JAR filename changed**: The default name may be different; use `<bootable-jar-name>` to customize if needed

Be careful and precise with XML formatting. Preserve indentation and structure.
