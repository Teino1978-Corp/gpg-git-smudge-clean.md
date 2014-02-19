Generate a GPG Key and revocation cert per http://www.gnupg.org/gpg/en/manual.html:


```
gpg --key-gen
gpg --output revoke.asc --gen-revoke <my user ID or email>
```

Once gpg key in keyring we can encrypt and decrypt files.


TODO: create second user and add their public key to keyring to use it for encryption


Update `.git/config` to clean files when they're added to staging by encrypting them and to smudge them by decrypting (see figures 7.2 and 7.3 from http://git-scm.com/book/ch7-2.html):

```
[filter "gpg"]
        clean = "gpg --encrypt --recipient <user ID or email of user who needs the secret>"
	smudge = "gpg --decrypt"
```

Tell git which files are secrets in `.gitattributes`:

```
*.secret filter=gpg
```


Example:


```shell
$ git init gpg-smudge-test
$ cd gpg-smudge-test
$ echo -n '[filter "gpg"]
        smudge = "gpg --decrypt"
        clean = "gpg --encrypt --recipient me@gmail.com"
' >> .git/config
$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[filter "gpg"]
	smudge = "gpg --decrypt"
	clean = "gpg --encrypt --recipient me@gmail.com"
$ echo '*.secret filter=gpg' > .gitattributes 
$ cat .gitattributes 
*.secret filter=gpg
$ echo '42' > secret.secret
$ cat secret.secret 
42
$ git add secret.secret
$ git diff # no diff since it's an binary file
$ git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#	new file:   secret.secret
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	.gitattributes
$ git commit -vm 'Add secret secret' 
[master (root-commit) 6c4c5c7] Add secret secret
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 secret.secret
$ git rev-list --objects -g --no-walk --all 
6c4c5c7bfd54b63244437af923e640511b79984c
85a36f2a401ea995cc48bfda0ff490b3a505c98b 
f814594185aa3462ef7a7cc3bd2ba2b9595c4ee9 secret.secret
$ git cat-file -p f814594185aa3462ef7a7cc3bd2ba2b9595c4ee9 
�
  GP*qj�x��A
              <���Z$��˂L�u<�ʜ��\�$e���� g=����)��fڏ�3���j}pぎ 6�������o���h�h)��`�]-*P
                                                                                          �
                                                                                           �b�eFN���ch�0����rl����&1@Z�'��5��5�WoW�5y������2��@�%=j��n�Ӕxi;b}��Fz����5 E����<�ko�a3�"Dh��O�{�k��㠢5�id-Lh
\+�E:����0b�>�����������{�O�i�O��59ӫ���<���\��mR�Yz�*>�y0~��b�% 
$ cat secret.secret
42
$ echo 'is 7 * 6' >> secret.secret 
$ cat secret.secret 
42
is 7 * 6
$ git add secret.secret
$ git commit -vm 'Update secret secret' 
[master b7a7ea4] Update secret secret
 1 file changed, 0 insertions(+), 0 deletions(-)
$ git rev-list --objects -g --no-walk --all 
b7a7ea4e47982336fd7f3ce21069f287a5d3190e
5995b4866b48a344ecb25155555eb19423e5c40a 
467ef996090ab76ea5dd2cf826940cbdf2576f48 secret.secret
$ git cat-file -p 467ef996090ab76ea5dd2cf826940cbdf2576f48 
�
  GP*qj�x�
�͋l��$[o4�»�MG��Kj��	�p�c~�ʆ����
                                    '�/[ʙi�$O���o*+Eod
                                                       Q ��l�*{
�����VkH�
             �U 0��>���lR
Z�>�j�Ƽ~���`���-h���T����=|`�2[��(���gX�-�wF�"��)z�]V;�,/jm���ڧ5�L�ֻ�TY�CF}���;6��/���PИW�h��Z|��$�_P'��7 8ڌ���o*�G]+�sC	+6jħ
pC��7k �~V�z���`'d}�Gxd���g�R�ß���Ej1mQv52��
                                               ^�%
$ git checkout HEAD~1

You need a passphrase to unlock the secret key for
user: "Greg G <me@gmail.com>"
2048-bit RSA key, ID 6AB47893, created 2014-02-19 (main key ID 9C54783A)

gpg: encrypted with 2048-bit RSA key, ID 6AB47893, created 2014-02-19
      "Greg G <me@gmail.com>"
Note: checking out 'HEAD~1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 6c4c5c7... Add secret secret
$ cat secret.secret                                                                                   6c4c5c7 
42
```


Note: 

* Each changes produces a completely new encrypted file
* Length of encrypted file leaks length of decrypted file