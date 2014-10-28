# Closure Compiler Maven Artifacts

# Introduction

The closure compiler is available in the Maven Central Repository. 

We currently do not have an official plugin for compiling your JS with Maven, but there should be some unofficial plugins on the RelatedProjects page.

# Usage

To include closure-compiler in your project add the following to your build configuration:

## Maven

      <dependency>
          <groupId>com.google.javascript</groupId>
         <artifactId>closure-compiler</artifactId>
         <version>v20141023</version>
      </dependency>

## Apache Ivy:

      <dependency org="com.google.javascript" name="closure-compiler" rev="v20141023"/>

## Groovy Grape:

      @Grapes(
          @Grab(group='com.google.javascript', module='closure-compiler', version='v20141023')
      )

## Apache Buildr

      'com.google.javascript:closure-compiler:jar: v20141023'


# Release Archive

Releases before v20130227 were based on the subversion commit number.  The following table identifies the matching date-based releases.

<table>
  <tr><td>r2388</td><td>Corresponds to the 20121212 release</td></tr>
  <tr><td>r2180</td><td>Corresponds to the 20120917 release</td></tr>
  <tr><td>r2079</td><td>Corresponds to the 20120711 release</td></tr>
  <tr><td>r1918</td><td>Corresponds to the 20120430 release</td></tr>
  <tr><td>r1810</td><td>Corresponds to the 20120305 release</td></tr>
  <tr><td>r1741</td><td>Corresponds to the 20112023 release</td></tr>
  <tr><td>r1592</td><td>Corresponds to the 20111114 release</td></tr>
  <tr><td>r1459</td><td>Corresponds to the 20111003 release</td></tr>
  <tr><td>r1352</td><td>Corresponds to the 20110811 release</td></tr>
  <tr><td>r946</td><td>Corresponds to the 20110405 release</td></tr>
  <tr><td>r916</td><td>Corresponds to the 20110322 release</td></tr>
  <tr><td>r706</td><td>Corresponds to the 20110119 release</td></tr>
  <tr><td>r606</td><td>Initial Mavenized Release</td></tr>
</table>