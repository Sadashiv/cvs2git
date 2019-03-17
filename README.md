CVS repo migration with history to GIT
======================================

Refrance link: http://clusterfrak.com/devops/git/git_cvs2git/

CVS is one of the first major Source Code Management tools that was available and widely used.
Information about it can be found here. In this tutorial we will be covering the process of 
converting a CVS repository over to git, a more modern, more powerful SCM.

1. Prerequisties:
  I. CentOS6 or RHEL6

2. Create below folder structure

  mkdir /{cvs,git}
  mkdir /tmp/cvs_migration

  I.   /cvs will be used to hold the rsync or copy(with history) clone of the cvs repository
  II.  /git will be used to hold the converted project(s).
  III. /tmp/cvs_migration will be used during the conversion, data will be extracted from /cvs,
       pulled into git_migration and used to create conversion logs, 
       which will be used to populate the final converted project(s) in /git.

3. Install Epel:
 To ensure that we get the latest packages that we need for git, svn, and cvs install the epel repository.

 yum install epel-release
 yum install git #Install git
 yum install svn #Install SVN
 yum install rcs cvs #Install rcs and CVS
 #The Revision Control System (RCS) manages multiple revisions of files
 #CVS Concurrent version system

4. Supporting tool installation
 yum install openssh-clients rsync wget

5. CVS2SVN:
  The last tool that we need is the cvs2svn.
  This will be the tool that we use to do the conversion of the cvs repo over to git format.

  cd /tmp
  wget ftp://ftp.pbone.net/mirror/dag.wieers.com/redhat/el6/en/x86_64/dag/RPMS/cvs2svn-2.3.0-1.el6.rf.noarch.rpm
  rpm -ivh cvs2svn-2.3.0-1.el6.rf.noarch.rpm

  Note: cvs2svn rpm installation may fail due to rpmforge-release package
        Download and install "rpmforge-release" package
        wget -c http://ftp.tu-chemnitz.de/pub/linux/dag/redhat/el6/en/i386/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.i686.rpm
        rpm -ivh rpmforge-release-0.5.3-1.el6.rf.i686.rpm
  All dependent package installed

  rpm -ivh cvs2svn-2.3.0-1.el6.rf.noarch.rpm

6. This crucial and important step to proceed the migration
    Login to CVS server and checkout repo
    export CVSROOT=:pserver:user:password@IP:/cvs/reponame
    cvs -d :pserver:user:password@IP:/cvs/reponame checkout -r HEAD Modulename
    Since CVS repo checkout will not bring history with CVSROOT folder missing you will not able to migrate to git

    Alternative way is to zip or gz /cvs/reponame/modulename /cvs/reponame/CVSROOT
    copy zip or gz to remote server
    scp reoname.zip user@IP:/cvs

7. Login to server cvs2svn package installed
   cd /cvs
   unzip -r reoname.zip
   cvs2git --blobfile=/git/git-blob.dat --dumpfile=/git/git-dump.dat --username=Sadashiv
   --fallback-encoding=ascii reponame >> /tmp/cvs_migration/reponame.log

   NOTICE:
   This step could take several hours depending on both the size of the repository and the number of commits.

8. Init the git repo, and import the converted cvs repo from the conversion created blob files: 

    cd /git
    git init --bare reponame.git
    cd project.git
    cat /git/git-blob.dat /git/git-dump.dat | git fast-import
    git gc --prune=now

9. Run the CVS migration for a several projects:
    If you have multiple projects, then you can start the process on all of them at once using the following methodology.

    for dir in /cvs/*;
    do cd /cvs
    PROJECT=`echo $dir | awk -F'/' {'print $4'}`;
    cvs2git --blobfile=/git/git-blob.dat --dumpfile=/git/git-dump.dat --username=myusername \
    --fallback-encoding=ascii $dir >> /tmp/cvs_migration/$PROJECT.log;
    cd /git;
    git init --bare $PROJECT.git;
    cd $PROJECT.git;
    cat /git/git-blob.dat /git/git-dump.dat | git fast-import;
    git gc --prune=now;
    done


10. Push the repo to Git:
    The last and final step of the migration is to push the new repository or repositories to Git.
    Depending on the git server / service that you are using, you will either have to create a new 
    repo for each of your converted cvs repos, or use the service's tools to do a mass import.

    Pushing repos manually:

    cd /git/project.git
    git remote add origin http://gitlab.yourcompany.com/namespace/git_repo_name.git
    git config --global user.email "userid@yourcompany.com"
    git config --global user.name "UserName"
    git add --all
    git commit -m "Conversion from CVS to Git"
    git push origin master



Feel free contact for any queries on cvstogit migration
Email id: sadamsrit@gmail.com








