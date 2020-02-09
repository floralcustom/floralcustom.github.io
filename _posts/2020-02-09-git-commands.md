# Git Commands

# 🚚 Run Locally

Frequently used commands. This is a helpful page to Git or Github.

In the `git bash` , 사용자 이름과 메일 설정:

     git config --global user.name "HJ"
     git config --global user.mail "floralcustom@gmail.com"

Config check, use:

    git config --list
    git config user.name
    git config --show-origin user.name //최초 설정 확인
    

End of line :

    시스템설정대로 : git config —global core.eof native
    시스템설정대로 : git config —global core.autcrlf false

Diff result Highlighting :

    git config -global pager.diff ‘diff-highlight | less’

Create Local repository :

    mkdir project
    cd project
    git init
    git add *
    git commit -m “My first version managing directory”

저장소 내려받기 :

    git clone id@원격저장소_URL
    git clone /로컬저장소/저장소

File 상태 확인(Tracked : Modified/Unmodified/Staged, Untracked : Unstaged) :

    git statusgit status -s :
    : OO filename
    : First O - Staging 상태를 보여줌
    : Second O - Working directory 상태를 보여줌
    : M - Modifyed
    : A - Added
    : ? - Unstaged
    Staging Area file을 Unstage로 변경하기
        git reset HEAD filename
        git checkout — filename
