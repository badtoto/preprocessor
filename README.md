#Preprocessor - A Preprocess Tool for Java

###It based on ant copy task.
- supports the following files:
- Java/JSP
- HTML/javascript/CSS
- XML
- Properties files
- supports file encoding specified
- supports filtering


###Sample.java

        //#ifdef CUSTOMER1
        private String abc = "abc";
        //#else
        private String abc = "cba";
        //#endif
        
        
###build.xml

        <project name="sample.project" default="war" basedir=".">
        
                <!-- Initialize -->
                <!-- Pre-processor output directory -->
                <property name="todir" value="standard" />
                <!-- Set the Project Name -->
                <property name="CUSTOMER1" value="defined" />
        
                <patternset id="none.customer.file">
                        <!--Except Customer1 -->
                        <exclude name='**/Customer1.java' />
                </patternset>
        
                <!-- Environment -->
                <property name="catalina.home" value="D:/tomcat/apache-tomcat-6.0.16"/>
                <property name="appname" value="project1" />
                <property name="deploy.name" value="deploy" />
                <property name="deploy.dir" value="${basedir}/${deploy.name}" />
                <property name="war.name" value="${appname}_${todir}" />
                <property name="jar.name" value="project1" />
                <property name="release-version" value="2.3" />
        
                <!-- Debug parameter -->
                <property name="compile.debug" value="true"/>
                <property name="compile.deprecation" value="false"/>
                <property name="compile.optimize" value="true"/>
        
                <!-- Tomcat Manager parameter -->
                <property name="tomcat.manager.url" value="http://localhost:8080/manager"/>
                <property name="tomcat.manager.username" value="admin"/>
                <property name="tomcat.manager.password" value="admin"/>
        
                <path id="tomcat.classpath">
                        <fileset dir="${catalina.home}/lib">
                                <include name="*.jar/" />
                        </fileset>
                        <fileset dir="${catalina.home}/bin">
                                <include name="*.jar/" />
                        </fileset>
                        <pathelement path="${catalina.home}/lib/catalina-ant.jar" />
                </path>
        
                <path id="compile.classpath">
                        <!-- lib path of tomcat -->
                        <fileset dir="${catalina.home}/lib">
                                <include name="*.jar"/>
                        </fileset>
        
                        <!-- project lib path -->
                        <fileset dir="${deploy.dir}/${todir}/WEB-INF/lib">
                                <include name="*.jar"/>
                        </fileset>
                </path>
        
                <!-- Clean the old output -->
                <target name="clean" >
                        <delete file="${deploy.dir}/${war.name}.war" description="Delete the old WAR file"/>
                        <delete dir="${deploy.dir}/${todir}" description="Delete the old directory"/>
                </target>
        
                <!-- Pre-process the files with filtering. Everything instance -->
                <taskdef name="preprocess" classname="preprocessor.ant.PreprocessTask" classpath="D:/java-libs/preprocessor.jar" />
                <target name='preprocess' depends="clean">
                        <!-- Set the filter -->
                        <filter token="app.version" value="${release-version}"/>
                        <!-- Begin preprocess -->
                        <preprocess todir='${deploy.dir}/${todir}' filtering='true'>
                                <fileset dir='.'>
                                        <exclude name='${deploy.name}/**' />
                                        <exclude name='.settings/**' />
                                        <exclude name='work/**' />
                                        <exclude name='**/*.jar' />
                                        <exclude name='**/classes/**' />
                                        <exclude name='**/images/*' />
                                        <exclude name='**/CVS/**' />
                                        <exclude name='**/.*' />
                                        <exclude name='*build.xml' />
                                        <!-- except encoding path -->
                                        <exclude name='pc/**' />
                                        <!-- except customer files -->
                                        <patternset refid="none.customer.file"/>
                                </fileset>
                        </preprocess>
                        <!-- preprocess encoding specified path -->
                        <preprocess todir='${deploy.dir}/${todir}' filtering='true' inputencoding="utf-8" outputencoding="utf-8">
                                <fileset dir='.'>
                                        <exclude name='${deploy.name}/**' />
                                        <include name='pc/**' />
                                        <exclude name='**/images/*' />
                                </fileset>
                        </preprocess>
                        <!-- Copy images , no filter-->
                        <copy todir="${deploy.dir}/${todir}" filtering="false" >
                                <fileset dir=".">
                                        <exclude name='${deploy.name}/**' />
                                        <include name='WEB-INF/lib/*' />
                                        <include name='**/images/*' />
                                </fileset>
                        </copy>
                </target>
        
                <!-- Compile Java classes -->
                <target name="compile" depends="preprocess" description="Compile Java sources">
                        <mkdir dir="${deploy.dir}/${todir}/WEB-INF/classes"/>
                        <javac srcdir="${deploy.dir}/${todir}/WEB-INF/src"
                          destdir="${deploy.dir}/${todir}/WEB-INF/classes"
                            debug="${compile.debug}"
                      deprecation="${compile.deprecation}"
                         optimize="${compile.optimize}">
                                <classpath refid="compile.classpath"/>
                        </javac>
        
                        <copy todir="${deploy.dir}/${todir}/WEB-INF/classes">
                                <fileset dir="${deploy.dir}/${todir}/WEB-INF/src" excludes="**/*.java"/>
                        </copy>
                </target>
        
                <!-- make jar file -->
                <target name="jar" depends="compile">
                        <jar compress="true" destfile="${deploy.dir}/${todir}/WEB-INF/lib/${jar.name}-${release-version}.jar">
                                <fileset dir="${deploy.dir}/${todir}/WEB-INF/classes">
                                        <include name="**" />
                                </fileset>
                        </jar>
                </target>
        
                <!-- make war file -->
                <target name="war" depends="clean, jar" description="Build all the files and package them into a WAR file">
                        <war warfile="${deploy.dir}/${war.name}.war" 
                                webxml="${deploy.dir}/${todir}/WEB-INF/web.xml" compress="true" update="true" >
                                <webinf dir="${deploy.dir}/${todir}/WEB-INF">
                                        <include name="**" />
                                        <exclude name="web.xml" />
                                        <exclude name="classes/**" />
                                        <exclude name="lib/**" />
                                        <exclude name="src/**" />
                                </webinf>
                                <lib dir="${deploy.dir}/${todir}/WEB-INF/lib">
                                        <include name="**" />
                                </lib>
                                <fileset dir="${deploy.dir}/${todir}">
                                        <include name="**" />
                                        <exclude name="WEB-INF/**" />
                                        <exclude name=".settings/**" />
                                        <exclude name=".classpath" />
                                        <exclude name=".project" />
                                        <exclude name=".tomcatplugin" />
                                        <exclude name="*build.xml" />
                                </fileset>
                        </war>
                </target>
        
                <!-- deploy project to server -->
                <taskdef name="deploy" classname="org.apache.catalina.ant.DeployTask" classpathref="tomcat.classpath" />
                <target name="deploy" description= "Deploy application in Tomcat">
                        <deploy 
                                url="${tomcat.manager.url}"
                                username="${tomcat.manager.username}"
                        password="${tomcat.manager.password}"
                        path="/${war.name}"
                        war="file:${deploy.dir}/${war.name}.war"
                        />
                </target>
        
                <!-- undeploy project to server -->
                <taskdef name="undeploy" classname="org.apache.catalina.ant.UndeployTask" classpathref="tomcat.classpath" />
                <target name="undeploy" description= "Undeploy application in Tomcat">
                        <undeploy 
                                url="${tomcat.manager.url}"
                                username="${tomcat.manager.username}"
                        password="${tomcat.manager.password}"
                        path="/${war.name}"
                        />
                </target>
        
        </project>
