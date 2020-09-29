Copy and Paste into AppleWin's debugger running _Nox Archaist 0.8.9.0-beta_

```asm
// Version 0.8.9.1 Alpha
8668:18     // [19   ]
8677:F8 40  // [25 7A]

                                //  ; ----------
                                //  PrintName.PushXY
4078:8E 7F 40                   //                  STX SH.4178.TempX
407B:8C 7F 42                   //                  STY SH.41F8.TempY
407E:60                         //                  RTS
407F:00                         //  SH.4178.TempX   DB $00

                                //  ; ====================
                                //  PrintName.Main
40F8:20 78 40                   //                  JSR PrintName.PushXY
40FB:20 F8 42                   //                  JSR PrintName.PrefixChar
40FE:F0 78                      //                  BEQ .PrintName.Main2        ; Always ***See fake BRA [1]***
                                //  .PrintName.Main2
4178:20 25 7A                   //                  JSR PRINT.STR
417B:20 F8 47                   //                  JSR PrintName.RestoreCharType
417E:90 78                      //                  BCC .PrintName.Main3        ; Always ***See fake BRA [2]***
                                //  .PrintName.Main3
41F8:20 78 43                   //                  JSR PrintName.PostfixChar   ; returns A=0
41FB:8D 7E 2F                   //                  STA COUT_CHAR_TYPE          ; emulate COUT and PRINT.STR which reset back to NORMAL
41FE:F0 78                      //                  BEQ PrintName.PopXY         ; Always ***see fake BRA [1]***

                                //  ; ----------
                                //  PrintName.PopXY
4278:AE 7F 40                   //                  LDX SH.4178.TempX
427B:AC 7F 42                   //                  LDY SH.41F8.TempY
427E:60                         //                  RTS
427F:00                         //  SH.41F8.TempY   DB $00

                                //  ; ----------
                                //  PrintName.PrefixChar
42F8:20 78 47                   //                  JSR PrintName.isINVERSE     ; C=0 Normal, C=1 Inverse
42FB:69 00                      //                  ADC #00                     ; A=0 Normal, A=1 Inverse 
42FD:4C 78 44                   //                  JMP PrintName.Glyph
                                //  ; ----------
                                //  PrintName.PostfixChar
4378:A5 24                      //                  LDA HTAB
437A:C9 27                      //                  CMP #$27                    ; At right border?
437C:90 7A                      //                  BCC PrintName.PostfixChar2  ; PrintName.PostfixChar2
437E:60                         //                  RTS                         ; Nothing to draw
437F:EA                         //                  NOP                         ; pad to mod 8
                                //  PrintName.PostfixChar2
43F8:20 78 47                   //                  JSR PrintName.isINVERSE     ; C=0 Normal, C=1 Inverse
43FB:69 02                      //                  ADC #02                     ; A=2 Normal, A=3 Inverse 
43FD:4C 78 44                   //                  JMP PrintName.Glyph         ; **INTENTIONAL FALL INTO

                                //  ; ----------
                                //  PrintName.Glyph                             ; C=?   A=abcdefgh
4478:4A                         //                  LSR                         ; C=h   A=0abcdefg ***NOTE: make positive
4479:AA                         //                  TAX
447A:A9 00                      //                      LDA #0                  ; C=h   A=00000000
447C:6A                         //                      ROR                     ;       A=h0000000
447D:18                         //                      CLC
447E:90 78                      //                      BCC .PrintName.Glyph2   ; Always
                                //  .PrintName.Glyph2
44F8:69 78                      //                      ADC #$78                ;      A=$78 or $F8; AddrLo = h*$80 - $8
44FA:85 E0                      //                      STA HRCG.SHAPE.OFFSET
44FC:8A                         //                  TXA                         ; Addr = $5000 + (abcdefg)
44FD:10 79                      //                  BPL .PrintName.Glyph3       ; Always; COUT Extended char < $7F
44FF:EA                         //                  NOP                         ; pad to mod 8
                                //  .PrintName.Glyph3
4578:69 48                      //                  ADC /ExtendedFont
457A:85 E1                      //                  STA HRCG.SHAPE.OFFSET+1
457C:A0 00                      //                  LDY #0                      ; Copy Extended Glyph to HGR.BUFFER
457E:F0 78                      //                  BEQ PrintName.CopyGlyph     ; Always; INTENTIONAL FALL INTO
                                //  PrintName.CopyGlyph
45F8:B1 E0                      //                  LDA (HRCG.SHAPE.OFFSET),Y
45FA:99 F8 22                   //                  STA HRCG.BUFFER,Y
45FD:C8                         //                  INY
45FE:D0 78                      //                  BNE .PrintName.CopyGlyph1   ; Always
                                //  .PrintName.CopyGlyph1
4678:C0 08                      //                  CPY #8
467A:B0 7C                      //                  BCS .PrintName.CopyGlyph2
467C:4C F8 45                   //                  JMP PrintName.CopyGlyph
467F:EA                         //                  NOP                         ; pad to mod 8
                                //  .PrintName.CopyGlyph2
46F8:20 07 03                   //                  JSR COUT.CUSTOM.USE.HTAB
46FB:E6 24                      //                  INC HTAB
46FD:A9 00                      //                  LDA #00                     ; ***[1] Fake BRA***
46FF:60                         //                  RTS

                                //  ; ----------
                                //  ; ON EXIT:
                                //  ;   C=0 if NORMAL
                                //  ;   C=1 if INVERSE
                                //  ; ----------
                                //  PrintName.isINVERSE
4778:AD 7E 2F                   //                  LDA COUT_CHAR_TYPE          ; COUT_CHAR_TYPE ($00 = normal, $7F = inverse)
477B:6A                         //                  ROR
477C:A9 00                      //                  LDA #0
477E:60                         //                  RTS                         ; Return C=0 or 1
477F:EA                         //                  NOP                         ; pad to mod 8
                                //  ; ----------
                                //  PrintName.RestoreCharType                   ;***HACK: Work around PRINT.STR resetting COUT_CHAR_TYPE to $00
47F8:AD 7B 22                   //                  LDA COUT_CHAR_TYPE.SAVED 
47FB:8D 7E 2F                   //                  STA COUT_CHAR_TYPE
47FE:18                         //                  CLC                         ; ***[2] Fake BRA***
47FF:60                         //                  RTS

                                //  ;*** NOTE: Must be PAGE aligned to $xx78!
                                //  ; The inverse glyphs have the inverse flag "inlined"
                                //  ;     $E1 -> $9E due to inverse flag is $7F thus E1 XOR 7F = 9E
                                //  ;     $03 -> $7C due to inverse flag is $7F thus 03 XOR 7F = 7C   ; Normally PRINT.STR resets inverse flag but we preserve it
                                //  ExtendedFont
4878:81 81 81 81 81 81 81 81    //              .HS 81.81.81.81.81.81.81.81     ; 00 NORMAL : Blue Border
48F8:9E 9E 9E 9E 9E 9E 9E 9E    //              .HS 9E.9E.9E.9E.9E.9E.9E.9E     ; 01 INVERSE: Blue Border with 2px inverse on right side
4978:00 00 00 00 00 00 00 00    //              .HS 00.00.00.00.00.00.00.00     ; 02 NORMAL : Space
49F8:7C 7C 7C 7C 7C 7C 7C 7C    //              .HS 7C.7C.7C.7C.7C.7C.7C.7C     ; 03 INVERSE: Space       with 2px inverse on left  side
```

