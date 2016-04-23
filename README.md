[![Build Status](https://travis-ci.org/zoffixznet/perl6-Audio-MIDI-Note.svg)](https://travis-ci.org/zoffixznet/perl6-Audio-MIDI-Note)

# NAME

Audio::MIDI::Note - playable MIDI note

# NON Perl 6 REQUIREMENTS

This module uses [Audio::PortMIDI](http://modules.perl6.org/repo/Audio::PortMIDI)
under the hood, which requires [portmidi C library](http://portmedia.sourceforge.net/portmidi/).
On Debian and derivatives, you can install it with `sudo apt-get install libportmidi0`

# SYNOPSIS

## *Canon in D* by Johann Pachelbel

Replication of [this sheet music](http://www.8notes.com/scores/420.asp).
Note: on my system, the midi player on that page kills timidity and I have
to do `sudo service timidity restart` to get my scripts to work again...

```perl6
    use Audio::MIDI::Note;

    my $stream = Audio::PortMIDI.new.open-output: 3, 32;
    END { $stream.close }

    my Audio::MIDI::Note $note .= new: :20tempo :$stream, :value(½), :49velocity;

    # Pachelbel `Canon in D`
    # Comments reference this sheet music: http://www.8notes.com/scores/420.asp
    $note   .rplay(<C4 E4>).rplay(<G3 D4>)  # first line of bars, with one repeat
            .rplay(<A3 C4>).rplay(<E3 B3>)
            .rplay(<F3 A3>).rplay(<C3 G3>)
            .rplay(<F3 A3>).rplay(<G3 B3>)
    for ^2;

    $note   .rplay(<C4 G4 E5>).rplay(<G3 B4 D5>) # second line of bars
            .rplay(<A3 C5   >).rplay(<E3 G4 B4>)
            .rplay(<F3 C4 A4>).rplay(<C3 E4 G4>)
            .rplay(<F3 F4 A4>).rplay(<G3 D4 B4>)

            # first two notes of the chord are half-notes and third one is a crotchet,
            # so we play the half-notes in async with .play, and then do
            # the crotchet series with blocking .rplay
            .velocity(64).value(¼) # play louder and switch to quarter note default
            .play(<C4 E4>, ½).rplay('C5').rplay('C5')
            .play(<G3 D4>, ½).rplay('D5').rplay('B4')

            # 10th bar
            .play(<A3 C4>, ½).rplay('C5').rplay('E5')
            .play('E3',    ½).rplay('G5').rplay('G4')

            # 11th and 12th bars
            .play(<F3 A3>, ½).rplay('A4').rplay('F4')
            .play('C3'   , ½).rplay('E4').rplay('G4')
            .play(<F3 A3>, ½).rplay('F4').rplay('C5')
            .play(<G3 B3>, ½).rplay('B5').rplay('G4')

            # 13th bar; after the first chord, we're asked to play louder (velocity)
            .play(<C4 E4>, ½).rplay('C5')
            .velocity(80)
            .rplay('E5', ⅛).rplay('G5', ⅛).rplay('G5', ⅛)
            .rplay('A5', ⅛).rplay('G5', ⅛).rplay('F5', ⅛)

            .play(<A3 C4>, ½).rplay('E5', ¼+⅛).rplay('E5', ⅛)
            .play(<E3 G3>, ½).rplay('E5',   ⅛).rplay('F5', ⅛).rplay('E5', ⅛).rplay('D5', ⅛)

            # 15th, 16th bar
            .play(<F3 A3>, ½).rplay('C5', ⅛).rplay('Bb4', ⅛).rplay('A4', ⅛).rplay('Bb4', ⅛)
            .play(<C3 E3>, ½).rplay('G4').rplay('E4')
            .play(<F3 A3>, ½).rplay('C4').rplay('F4', ⅛).rplay('E4', ⅛)
            .play(<G3 B3>, ½).rplay('D4').rplay('G4', ⅛).rplay('F4', ⅛)

            # 17th bar: we'll sound half-notes in async, and will use .rest
            # to play the quarter-note rest on the treble clef.
            # .rest can take a rest value as argument, but our current value is
            # already a crotchet, so no argument is needed:
            .play(<C4 E3>, ½).rest.velocity(64).rplay('C5')
            .play('G3', ½).rplay('D5').rplay('B4')

            # Last row of bars
            .play(<A3 C4>, ½).rplay('C5').rplay('E4')
            .play(<E3 B3>, ½).rplay('G4', ¼+⅛).rplay('A4', ⅛)
            .play(<F3 A3>, ½).rplay('F4').rplay('C4')
            .play(<C3 G3>, ½).rplay('E4').rplay('G4')
            .play(<F3 A3>, ½).rplay('F4').rplay('E4')
            .play(<G3 B3>, ½).rplay('D4').rplay('G4')
            .rplay(<C3 C4 E3>, 1)
    ;
```

## *Gorgoroth - A World to Win* Solo

Part of the solo from [A World to Win](https://www.youtube.com/watch?v=7EvOTkEMlug),
showing swap of instruments, re-use of repeating pieces of music, calculation
of triplet note values, and use of on- and off- beat velocity shortcuts.

```perl6
    use Audio::MIDI::Note;

    my $stream = Audio::PortMIDI.new.open-output: 3, 32;
    END { $stream.close }

    my Audio::MIDI::Note $note .= new: :31tempo :30instrument :$stream :value(⅔ * ⅛);

    # Looping `Gorgoroth - A World to Win` solo with organ chord in the background.
    # We use triplet notes and save repeating pieces into variables for reuse
    my &rhythm = *.rplay('D#5').rplay('D5').rplay('C5');
    my &riff = {
        .play(<C4 E4 G4>, 4, :19instrument, :40velocity)

        .rplay('C5',  ⅔*(¼+⅛) )
        .riff(&rhythm).rplay('C5',  ⅔*(¼+⅛) )
        .riff(&rhythm).rplay('G#4', ⅔*(¼+⅛) )
        .riff(&rhythm).rplay('A#4', ⅔*(¼+⅛) )
        .riff(&rhythm).rplay('C5',  ⅔*(¼+⅛) )
        .riff(&rhythm).rplay('C5',  ⅔*(¼+⅛) )
        .riff(&rhythm);
    };

    $note   .riff(&riff)
                .rplay('G5',  ¼, :on).rplay('F5', ¼, :off)
                .rplay('D#5', ¼     ).rplay('D5', ¼, :off)
            .riff(&riff)
                .rplay('F5', ¼, :on ).rplay('D#5', ¼, :off)
                .rplay('D5', ¼      ).rplay('A#4', ¼, :off)
    for ^10;
```

# DESCRIPTION

This module lets you play notes on a MIDI hardware or software device using
methods that allow to replicate sheet music.

# METHODS

## `new`

```perl6
my Audio::MIDI::Note $t .= new;
```

----

# REPOSITORY

Fork this module on GitHub:
https://github.com/zoffixznet/perl6-Audio-MIDI-Note

# BUGS

To report bugs or request features, please use
https://github.com/zoffixznet/perl6-Audio-MIDI-Note/issues

# AUTHOR

Zoffix Znet (http://zoffix.com/)

# LICENSE

You can use and distribute this module under the terms of the
The Artistic License 2.0. See the `LICENSE` file included in this
distribution for complete details.