#!/usr/bin/env perl
# this file is part of qubes-screenshooter, written by nil0x42
# check newer version @ https://github.com/nil0x42/qubes-screenshooter

use strict;
use warnings;

# - strip image's EXIF info
# - save screenshot on VM
# - load screenshot in VM clipboard
sub vm_screenshot {
    my $vm = shift;
    my $filepath = shift;
    my $filename;
    ($filename = $filepath) =~s/.*\///s;
    my $remotepath = "~/QubesIncoming/dom0/${filename}";

    # strip EXIF infos from image (OPSEC)
    system "mogrify -strip ${filepath}";

    # move image to VM
    system 'qvm-move-to-vm', $vm, $filepath;

    # send Qubes OS desktop notification
    system 'notify-send', 'Qubes Screenshot',
           "Screenshot sent to \'<b>$vm</b>\' VM\'s clipboard\n" .
           "<small><tt>~/QubesIncoming/dom0/<b>$filename</b></tt></small>";

    # xclip daemon keeps running until another process takes clipboard ownership
    exec ('qvm-run', '-q', '--', $vm,
           # "xdg-open ~/QubesIncoming/dom0/ & " .
          "printf ${remotepath} | xclip; " .
          "xclip -sel c -t image/png ${remotepath}"
          ) or die "qvm-run exec failed\n";
}

# let user choose target VM with `zenity`
sub choose_vm {
    return `qvm-ls | grep 'Running' | grep -Ev '^(dom0|sys-)' |
            cut -d' ' -f1 | sort | 
            zenity --list --height 300 \\
                   --title 'Screenshot to VM' \\
                   --text 'Select the VM to copy the screenshot' \\
                   --column 'Choose VM'`;
}

chomp(my $vm = choose_vm) or die;
vm_screenshot $vm, shift;
