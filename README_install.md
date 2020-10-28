Gentoo's selinux-hardened-refpolicy, tweaked for my needs ( https://gitweb.gentoo.org/proj/hardened-refpolicy.git/tree/ )

Since original file leaves much to be desired, especially on Gentoo systems that use systemd, I've added bunch of the missing stuff. Now, at least for me, machine boots and works normally in enforcing mode. With original policy, one has to make a million tweaks in million try-mand-test attempts, resulting in many additional modules, which create a mess. I used to have 60+ modules with extra stuff for systemd alone. Then there is additional stuff. GUI (Enlightenment or Sway) wouldn't even start with original policy. It died without an explanation. Now all this is in base policy. Loading bazzilion of extra modules was mess and slow. And remaining bugs in infrastructure kept demonstrating some weird bugs - some rules were silently dropped etc. I also added extra types that make stuff like firefox safer to use. NOw firefox doesn't need to access your home dir in general. There are two noeew filetypes just for download directory. One pair is for ~user/Downloads map and the other is for /home/000_DLD map. First one is meant for individual users, other is for stuff that is meant to be shared amongst users. Same with Skype etc.

I don't know if it's about the policy or changes in existing firefox modules etc, but now I can run Firefox and thunderbird in firejail ( makes containers for your browser and other exposed apps - highly recommended!).
Didn't yet got around to write a module for Skype ( don't use it much anyway) etc, but that will follow soon.

Instead of making extra modules, I added all the needed changes into policy itself. do a git diff master..my_policy to review them. Maybe someone sneaked something sinister, I was off my pills or there is some other gross mistake.

1.  	Copy the stuff in your workmap: git clone given_git_root /usr/src/MY_SELINUX_POLICY_WORKMAP

2.	cd /usr/src/MY_SELINUX_POLICY_WORKMAP ; git checkout brane_dev ;  make install

3.    	nano /etc/selinux/config ( SELINUXTYPE=my_policy) (SELINUX=enforcing)

4.	change kernel load line in grub by adding kernel options selinux=1 enforcing=0

5.	semodule -B ; semodule -R

6.	relabel the disk as suggested on Gentoo's page. I have something like this in my .bashrc ( before that, make the map /mnt/selinux_recontext) :

	SEL_ROOT=/etc/selinux/${SEL_NAME} 
	SEPOL_ROOT=${SEL_ROOT}/policy 
	SEPOL=${SEPOL_ROOT}/policy.${SEPOL_NAME} 
	SECON_ROOT=${SEL_ROOT}/contexts 
	SEFCON_ROOT=${SECON_ROOT}/files 
	SEFCON=${SEFCON_ROOT}/file_contexts

	function error_out() { echo "ERROR: ${1}" exit "${$2:- 1}" }

	function info_notice() { echo "INFO: ${1}" }

	function fcon_regen() { [ ! -d /mnt/selinux_recontext ] && { mkdir /mnt/selinux_recontext || \
	    error_out "fcon_regen: couldn't make initial mount_bind to /mnt/selinux_recontext" ; } 
	    umount /mnt/selinux_recontext 
	    rlpkg -a -r || info_notice "fcon_regen: rlpkg has ended, about to do setfiles..."; 
	    mount --bind / /mnt/selinux_recontext || error_out "fcon_regen: mount failed"; 
	    echo "**************************************************************************************************************" 
	    echo "doing setfiles for policy: ${SEL_NAME}" 
	    setfiles -F -r /mnt/selinux_recontext ${SEFCON} /mnt/selinux_recontext || error_out "fcon_regen: setfiles failed..."; 
	    umount /mnt/selinux_recontext || error_out "fcon_regen: final umount of /mnt/selinux_recontext failed..."; info_notice "relabel done..."; }
	    
	With that in .bashrc, unless I have overlooked something, you should be able to do complete relabelling of they system...
	    

    That's it. You should be able to reboot in permissive mode. AFter boot, go "setenforce 1" to try enforcing mode. 
    If/when that works, you can change boot parameter to enforcing=1 to enter the boot itself in enforcing mode and see if something explodes in boot sequence.
