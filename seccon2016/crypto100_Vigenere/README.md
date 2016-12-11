問題
===
    Vigenere
    k: ????????????
    p: SECCON{???????????????????????????????????}
    c: LMIG}RPEDOEEWKJIQIWKJWMNDTSR}TFVUFWYOCBAJBQ

    k=key, p=plain, c=cipher, md5(p)=f528a6ab914c1ecf856a1d93103948fe

     |ABCDEFGHIJKLMNOPQRSTUVWXYZ{}
    -+----------------------------
    A|ABCDEFGHIJKLMNOPQRSTUVWXYZ{}
    B|BCDEFGHIJKLMNOPQRSTUVWXYZ{}A
    C|CDEFGHIJKLMNOPQRSTUVWXYZ{}AB
    D|DEFGHIJKLMNOPQRSTUVWXYZ{}ABC
    E|EFGHIJKLMNOPQRSTUVWXYZ{}ABCD
    F|FGHIJKLMNOPQRSTUVWXYZ{}ABCDE
    G|GHIJKLMNOPQRSTUVWXYZ{}ABCDEF
    H|HIJKLMNOPQRSTUVWXYZ{}ABCDEFG
    I|IJKLMNOPQRSTUVWXYZ{}ABCDEFGH
    J|JKLMNOPQRSTUVWXYZ{}ABCDEFGHI
    K|KLMNOPQRSTUVWXYZ{}ABCDEFGHIJ
    L|LMNOPQRSTUVWXYZ{}ABCDEFGHIJK
    M|MNOPQRSTUVWXYZ{}ABCDEFGHIJKL
    N|NOPQRSTUVWXYZ{}ABCDEFGHIJKLM
    O|OPQRSTUVWXYZ{}ABCDEFGHIJKLMN
    P|PQRSTUVWXYZ{}ABCDEFGHIJKLMNO
    Q|QRSTUVWXYZ{}ABCDEFGHIJKLMNOP
    R|RSTUVWXYZ{}ABCDEFGHIJKLMNOPQ
    S|STUVWXYZ{}ABCDEFGHIJKLMNOPQR
    T|TUVWXYZ{}ABCDEFGHIJKLMNOPQRS
    U|UVWXYZ{}ABCDEFGHIJKLMNOPQRST
    V|VWXYZ{}ABCDEFGHIJKLMNOPQRSTU
    W|WXYZ{}ABCDEFGHIJKLMNOPQRSTUV
    X|XYZ{}ABCDEFGHIJKLMNOPQRSTUVW
    Y|YZ{}ABCDEFGHIJKLMNOPQRSTUVWX
    Z|Z{}ABCDEFGHIJKLMNOPQRSTUVWXY
    {|{}ABCDEFGHIJKLMNOPQRSTUVWXYZ
    }|}ABCDEFGHIJKLMNOPQRSTUVWXYZ{
    Vigenere cipher
    https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher
解
====
文字単位の encoder と総当たりコードを書いて

    SOURCE="ABCDEFGHIJKLMNOPQRSTUVWXYZ{}"
    CHARS = SOURCE.chars

    def encode(char, key)
      c = SOURCE.index(char)
      k = SOURCE.index(key)
      (SOURCE*2)[c+k]
    end

    def crack(char, chiper)
      (CHARS.select{|_| encode(char, _)==chiper})[0]
    end

ちまちま試してキーが

    VIGENER?????
    
であることを得る。残り５文字なので残りを総当たり。

    def decode(chiper, key)
      k = SOURCE.index(key)
      c = SOURCE.index(chiper)
      (SOURCE*2)[c-k]
    end

    def decode_str(chiper, key)
      chiper.chars.map.each_with_index{|_,i|
        k = (key*10)[i]
        if k == "*"
          "*"
        else
          decode(_, (key*10)[i])
        end
      }.join
    end

    require 'digest'
    def check(key)
      chiper = "LMIG}RPEDOEEWKJIQIWKJWMNDTSR}TFVUFWYOCBAJBQ"
      key = "VIGENER" + key
      ans = "f528a6ab914c1ecf856a1d93103948fe"
      plain = decode_str(chiper, key)
      md5 = Digest::MD5.hexdigest(plain)
      p([plain, key, md5])
      exit if md5==ans
    end

    CHARS.map{|c1|
      CHARS.map{|c2|
        CHARS.map{|c3|
          CHARS.map{|c4|
            CHARS.map{|c5|
              check [c1,c2,c3,c4,c5].join
            }
          }
        }
      }
    }
