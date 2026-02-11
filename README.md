# automatix

A remote script execution client/server using git to communicate.

The ideas is very simple: there is a main directory (let's say ~/automatix). Under it there is some subdirectories:

bin => repository for main scripts (mostly bash)
inbox => anything that is here may be used for a job
work => while executing a job, evey file for the job is moved to a subdirectory here with the job execution number
finished => when the job finishes, everything in the job execution subdirectory is moved here

There is a cli script that handle most of the work organizing the jobs, files for the job, etc. The name of the script is automatix.

# Automatix commands

'''
automatix new [--job NAME] [--move] main_script [additional_files ...]
'''

    Creates a new job using the filess passed through. The main file to be executed is the first file, but it may have more files.

    The options are:
        --job/-j => specifies a job name. Default to current date/time in YYYYMMDD-HHMMSS. If the name has the token _NOW_ it expandas the text _NOW_ to YYYYMMDD-HHMMSS.
        --move/-m => move the files instead of copying them

    The script does:
    1) updates the repository doing a git pull/rebase.
    2) creates a directory under inbox with the name of the job (or the date/time in YYYYMMDD-HHMMSS format) and copies all the files to this directory.
    3) copies/move all the files
    4) creates a .jobrc file in the job's directory (see .jobrc format)
    5) git all files in the directory & commit
    6) pushes it to the remote repository

'''
automatix update
'''
    1) updates the repository doing a git pull/rebase.
    2) do an "cleanup"
    3) if it has no job with a WAIT clause in working dir, get the first job in inbox (if any) and process it in background

'''
automatix process job_name
'''

    Process the job under jobname

    The script does:
    1) git mv the job directory (in inbox/job_name) to working directory
    2) cd to the directory
    4) source the .jobrc if exists
    5) initializes automatix.log with the start date of the job, and mark it's state as running.
    6) starts the $AUTOMATIX_COMMAND $AUTOMATIX_MAIN_SCRIPT redirecting all the output to automatix.out in background
    6) record the PID of the process in automatix.log
    7) waits AUTOMATIX_DELAY seconds
    8) add all files, commits and pushes
    9) fg the background call waiting it to finish
    10) when it finises append in the log the finish date, and the ERROR CODE
    11) git add -A ; git commit -m "finished $JOB" ; git push

'''
automatix cleanup
'''

    Check if there is some finished or aborted job.
    For each job in working directory:
        Check the automatix.log for the state:
            if it is running, get the PID, check if the pid is still running. If it is not running, mark it as aborted
            if it is running, leave it there. Add all files under it (specially log file and output), commit and push.
        If it is finished (or terminated), git mv it to the finished directory, commit and push.


# .jobrc format

The .jobrc is a simply bash script with variable definitions. The variables that are:

AUTOMATIX_COMMAND => The command to run (default: bash -c)
AUTOMATIX_MAIN_SCRIPT => the name of the main script (default: first file passed)
AUTOMATIX_WAIT => Should the server wait until it finishes to run the next job? (default: true)
AUTOMATIX_DELAY => number of seconds to wait before sending the first response (default 30)

