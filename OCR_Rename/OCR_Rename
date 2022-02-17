#!/bin/bash
# NOTE verion 1.3 #
FILES="./*.pdf" 
PATHS="./*/"

# Redirect both stdout and stderr to write to the debug logfile
exec 1>>./log/debug-log.txt 2>>./log/debug-log.txt

echo "Bash Script - $(date +%m-%d-%Y-%T) Creating OCR of $FILES" | tee -a ./log/OCRlog.txt


for f in $FILES; do
    # f drops prefixes, t adds .OCR.pdf to f so ocrmypdf can make a copy #
    f=${f##*/}
    t="${f%.pdf}.OCR.pdf"
    # checks to see if file already been processed. todo: detect if an OCR for that file exist #
     if [[ $f != *"OCR"* ]];then
     echo "OCR - $(date +%m-%d-%Y-%T) Starting to process file $f" | tee -a ./log/OCRlog.txt
     # ocr the file and rename it file.OCR.pdf: rotates, clean, removes background and skews #
     ocrmypdf -rc --remove-vectors "$f" "$t"
     else
     echo "$f File already OCR'd skipping" | tee -a ./log/OCRlog.txt
     fi
    # gets name for ocr'd file  #
    echo "OCR -$(date +%m-%d-%Y-%T) Processing $t file..." | tee -a ./log/OCRlog.txt
    # only reads first page and attempts to get the UN and Tool number #
    Usage=$(pdftotext -f 1 -l 1 -x 0 -y 0 -W 1000 -H 200 -nopgbrk -raw "$t" - | grep Usage | sed 's/[^0-9]//g' | sed -r 's/(.{5}).*/\1/')
    Serial=$(pdftotext -f 1 -l 1 -x 0 -y 0 -W 1000 -H 200 -nopgbrk -raw "$t" - | grep Serial | sed  -e 's/.*Serial//' -e 's/_//g' | awk '{for(i=1;i<=NF;i++){if($i~/-/){print $i}}}')
    # combines names to make file name #
    Fnewname="$Usage-$Serial"
     if
        #check if file already exist. so mv doesn't overwrite #
        echo "Re-name file - $(date +%m-%d-%Y-%T) Found text in $t" | tee -a ./log/OCRlog.txt 
        [ -n "${Fnewname##*_}" ];then
        echo "Re-name file - $(date +%m-%d-%Y-%T) Replacing $t with $Fnewname.pdf" | tee -a ./log/OCRlog.txt
         if [[ -f "$Fnewname.pdf" ]];then
           # making it log file can't be renamed  #
           echo "Re-name file - $(date +%m-%d-%Y-%T) Could not rename file $t to $Fnewname since name alreayd exist.Check for file in OCRfailedrenames.txt"
           echo "Re-name file - $(date +%m-%d-%Y-%T) OCR file $t was unable to be renamed due to $Fnewname already existing" >> OCRfailedrenames.txt
         else
           # renames file use -n just to be sure no files are overwritten #
           mv -n "$t" "$Fnewname".pdf
           echo "Re-name file - $(date +%m-%d-%Y-%T) File $Fnewname.pdf done" | tee -a ./log/OCRlog.txt
           echo "$(date +%m-%d-%Y-%T) $t rename to $Fnewname.pdf $(date +%m-%d-%Y-%T)" >> ./log/OCRrenames.txt
           # attempting to move files to new directory # 
           echo "Re-name file - $(date +%m-%d-%Y-%T) making folder $Usage and moving files into it"
           mkdir "$Usage"
           mv -n "$f" "$Fnewname.pdf" "./$Usage"
         fi
     else
        echo "Re-name file - $(date +%m-%d-%Y-%T) Could not extract text from $f,Either bad OCR read or file hasn't been OCR'd yet" | tee -a ./log/OCRlog.txt
     fi  
    # moves folders in disired 
    
    for p in $PATHS; do
       if [[ $p == ./$Usage/ ]]; then
         echo "File Movement - $(date +%m-%d-%Y-%T) $Serial" | tee -a ./log/OCRlog.txt       
         lastname=${Serial##*-}
         lastname=${lastname::3}
         echo "File Movement - $(date +%m-%d-%Y-%T) $lastname" | tee -a ./log/OCRlog.txt
         filename="${Serial%-*}-$lastname"
         echo "File Movement - $(date +%m-%d-%Y-%T) $filename" | tee -a ./log/OCRlog.txt
         filepath="$(find . -name "$filename"*)"
         echo "File Movement - $(date +%m-%d-%Y-%T) $filepath" | tee -a ./log/OCRlog.txt
         filepath=${filepath#*/}
         echo "File Movement - $(date +%m-%d-%Y-%T) $filepath" | tee -a ./log/OCRlog.txt
          if mv --backup=numbered "$Usage"/ "$filepath/" ; then 
             echo "File Movement - $(date +%m-%d-%Y-%T) $Usage containing $Serial moved to $filepath" | tee -a ./log/OCRlog.txt
          else mv --backup=numbered "$Usage"/ "./Requires_Attention/$Usage-$(date +%m-%d-%Y-%H-%M-%S)"
             echo "File Movement - $(date +%m-%d-%Y-%T) $Usage Folder already exist at $filepath moved folder into Requires_Attention" | tee -a ./log/OCRlog.txt
          fi   
       else 
         echo "File Movement - $(date +%m-%d-%Y-%T) $p did not work" | tee -a ./log/OCRlog.txt
       fi
     done  
done
echo "Finshed - $(date +%m-%d-%Y-%T) All files processed" | tee -a ./log/OCRlog.txt