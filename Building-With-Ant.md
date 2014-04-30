# How to use Ant to compile your JavaScript

As of the Feb 1, 2010 release, Closure Compiler has an [Apache Ant](http://ant.apache.org/) task implementation.

Here is an example of a build.xml file that uses Closure Compiler.

    <?xml version="1.0"?>
    <project basedir="." default="compile">
    
      <taskdef name="jscomp" classname="com.google.javascript.jscomp.ant.CompileTask"
               classpath="../build/compiler.jar"/>
    
      <target name="compile">
        
        <jscomp compilationLevel="simple" warning="verbose" 
    	    debug="false" output="output/file.js">
    
          <externs dir="${basedir}/src">
            <file name="extern.js"/>
          </externs>
    
          <sources dir="${basedir}/src">
            <file name="simple1.js"/>
            <file name="simple2.js"/>
          </sources>
    
          <sources dir="${basedir}/other">
            <file name="simple3.js"/>
          </sources>
    
        </jscomp>
        
      </target>
    
    </project>


New features are added occasionally, the [source for the task](http://code.google.com/p/closure-compiler/source/browse/#git%2Fsrc%2Fcom%2Fgoogle%2Fjavascript%2Fjscomp%2Fant) may be the best documentation.
 