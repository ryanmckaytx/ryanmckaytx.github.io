Title: Continous wc -l
Date: 2013-01-06 16:40
Author: Ryan McKay
Tags: awk, tail
Slug: continous-wc-l
Status: published

When you want to see how fast lines are being appended to a file, try this:  

> </p>
>
> <span style="font-family: Courier New, Courier, monospace; font-size: x-small;">tail -f \$filename \| awk '</span><span style="font-family: Courier New, Courier, monospace; font-size: x-small;">BEGIN {nl=0; format="%F %T"}</span><span style="font-family: Courier New, Courier, monospace; font-size: x-small;">{nl += 1; if (nl % 1000 == 0) print strftime(format) " " nl}'</span>

</p>

Just adjust the mod operand to how often you want a printout.
