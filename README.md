# dicom2nifti2minc

# Steps to Run the Pipeline

## Software Setup

To begin, please log onto Sockeye (https://ondemand.arc.ubc.ca/pun/sys/dashboard/) and open the Terminal window.

In the Terminal window above, please run the following code:

```sh
SCRATCH_DIR="/scratch/st-toddwood-1/$USER"

mkdir -p "$SCRATCH_DIR"

cd "$SCRATCH_DIR"

git clone -b dicom2nifti2minc_Version-1.00 --single-branch https://github.com/jshahki/dicom2nifti2minc.git

cd /scratch/st-toddwood-1/$USER/dicom2nifti2minc

```

Then run the following:

```sh

cd /scratch/st-toddwood-1/$USER/dicom2nifti2minc

bash ./code/dicom2nifti2minc_setup.sh

```

Please ensure to keep the Terminal window open while this process completes.

## Upload your subjects' DICOM data

After completing the software setup step, you can upload your data into the `/data` folder. Please create a new folder within `/data` for each subject you want to run. In each subjects' folder, place a single `.zip` folder containing the DICOM scans. Then, add your subjects that you would like to run in the `participants.tsv` list. Note that there isn't a required naming format, but just make sure that the name of the subject folders match with the name in the `participant.tsv` file. As well, make sure that there are no spaces in any of the subject/folder names.

For instance, your `/data` folder could look like this after uploading:

```
main_dicom2nifti2minc_directory
├──data
│   ├──participants.tsv
│   ├──sub-c01
│   │   └──SUBJECTC01.zip
│   ├──sub-c02
│   │   ├──SUBJECTC02.zip
│   └──sub-c03
│       └──SUBJECTC03.zip
```

And your corresponding `participants.tsv` file would look like the following:

```
participant_id
sub-c01
sub-c02
sub-c03
```

## Converting scans from DICOM to NIfTI

After uploading your data and listing your to-be-converted subjects in the participants.tsv file, please run the following code in the Terminal window:

```sh
cd /scratch/st-toddwood-1/$USER/dicom2nifti2minc
git pull

PARTICIPANTS_FILE="./data/participants.tsv"
SUB_SIZE=1  # Subjects per job

# Read subject IDs from file (ignore header)
mapfile -t SUBJECTS < <(tail -n +2 "$PARTICIPANTS_FILE" | cut -f1 | tr -d '\r' | sed '/^$/d')

N_SUBJECTS=${#SUBJECTS[@]}
ARRAY_LENGTH=$(( (N_SUBJECTS + SUB_SIZE - 1) / SUB_SIZE ))

echo "Found $N_SUBJECTS subjects. Launching array with $ARRAY_LENGTH jobs."

sbatch --array=0-$((ARRAY_LENGTH - 1)) ./code/dicom2nifti.sh
```

Once the conversion is completed, you can access each subjects' `.nii.gz` files in the `/derivatives` folder. Please rename your `.nii.gz` files if required before performing the next step.

## Converting scans from NIfTI to MINC

Once renaming is completed, please run the following code in the Terminal window:

```sh
cd /scratch/st-toddwood-1/$USER/dicom2nifti2minc
git pull

PARTICIPANTS_FILE="./data/participants.tsv"
SUB_SIZE=1  # Subjects per job

# Read subject IDs from file (ignore header)
mapfile -t SUBJECTS < <(tail -n +2 "$PARTICIPANTS_FILE" | cut -f1 | tr -d '\r' | sed '/^$/d')

N_SUBJECTS=${#SUBJECTS[@]}
ARRAY_LENGTH=$(( (N_SUBJECTS + SUB_SIZE - 1) / SUB_SIZE ))

echo "Found $N_SUBJECTS subjects. Launching array with $ARRAY_LENGTH jobs."

sbatch --array=0-$((ARRAY_LENGTH - 1)) ./code/nifti2minc.sh
```

Once complete, each subjects' `.mnc` files will be found in the `/derivatives` folder.
