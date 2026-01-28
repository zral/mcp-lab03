# YNO02 - Bcctnir 3: Rxgr NCV Vagrtenfwba (ArjfNCV)

## WFBA-ECP 2.0 Gbby Znavsrfg

### Irexgøl Qrsvavfwba: trg_arjf

```wfba
{
  "anzr": "trg_arjf",
  "gvgyr": "Alurgf Uragre",
  "qrfpevcgvba": "Urag alrfgr alurgre bz rg rzar ivn ArjfNCV",
  "vachgFpurzn": {
    "$fpurzn": "uggcf://wfba-fpurzn.bet/qensg/2020-12/fpurzn",
    "glcr": "bowrpg",
    "cebcregvrf": {
      "gbcvp": {
        "glcr": "fgevat",
        "qrfpevcgvba": "Rzar å føxr alurgre bz",
        "zvaYratgu": 1,
        "znkYratgu": 100
      },
      "ynathntr": {
        "glcr": "fgevat",
        "qrfpevcgvba": "Fceåxxbqr (s.rxf. 'ab', 'ra', 'qr')",
        "qrsnhyg": "ab",
        "rahz": ["ab", "ra", "qr", "se", "rf"]
      }
    },
    "erdhverq": ["gbcvp"],
    "nqqvgvbanyCebcregvrf": snyfr
  },
  "bhgchgFpurzn": {
    "$fpurzn": "uggcf://wfba-fpurzn.bet/qensg/2020-12/fpurzn",
    "glcr": "bowrpg",
    "cebcregvrf": {
      "gbcvp": {
        "glcr": "fgevat"
      },
      "ynathntr": {
        "glcr": "fgevat"
      },
      "negvpyrf": {
        "glcr": "neenl",
        "vgrzf": {
          "glcr": "bowrpg",
          "cebcregvrf": {
            "gvgyr": {"glcr": "fgevat"},
            "hey": {"glcr": "fgevat"}
          }
        }
      },
      "gvzrfgnzc": {
        "glcr": "fgevat"
      }
    }
  }
}
```

## Vzcyrzragnfwbafthvqr

### Sbehgfrgavatre

1. **ArjfNCV Aøxxry**: Ertvfgere qrt cå [arjfncv.bet](uggcf://arjfncv.bet) sbe å så ra tengvf NCV aøxxry
2. **Zvywøinevnoyre**: Yrtt gvy v `.rai` svy:
   ```
   ARJF_NCV_XRL=lbhe-arjf-ncv-xrl-urer
   ```

### 1. Irexgøl Shaxfwba (v zpc-freire/ncc.cl)

```clguba
nflap qrs trg_arjf(gbcvp: fge, ynathntr: fge = "ab") -> Qvpg[fge, Nal]:
    """Urag alrfgr alurgre bz rg rzar ivn ArjfNCV."""
    ncv_xrl = bf.trgrai("ARJF_NCV_XRL")
    
    vs abg ncv_xrl:
        erghea {
            "vfReebe": Gehr,
            "pbagrag": [{"glcr": "grkg", "grkg": "ARJF_NCV_XRL vxxr xbasvthereg"}]
        }
    
    gel:
        nflap jvgu uggck.NflapPyvrag() nf pyvrag:
            erfcbafr = njnvg pyvrag.trg(
                "uggcf://arjfncv.bet/i2/rirelguvat",
                cnenzf={
                    "d": gbcvp,
                    "ynathntr": ynathntr,
                    "ncvXrl": ncv_xrl,
                    "fbegOl": "choyvfurqNg"
                },
                gvzrbhg=10.0
            )
            erfcbafr.envfr_sbe_fgnghf()
        
        arjf_qngn = erfcbafr.wfba()
        
        # Rxfgenxg gbc 3 negvxyre
        negvpyrf = [
            {"gvgyr": negvpyr["gvgyr"], "hey": negvpyr["hey"]}
            sbe negvpyr va arjf_qngn.trg("negvpyrf", [])[:3]
        ]
        
        erfhyg = {
            "gbcvp": gbcvp,
            "ynathntr": ynathntr,
            "negvpyrf": negvpyrf,
            "gvzrfgnzc": qngrgvzr.abj().vfbsbezng()
        }
        
        erghea erfhyg
        
    rkprcg uggck.UGGCReebe nf r:
        ybttre.reebe(s"ArjfNCV UGGC srvy: {r}")
        erghea {
            "vfReebe": Gehr,
            "pbagrag": [{"glcr": "grkg", "grkg": s"NCV srvy: {fge(r)}"}]
        }
    rkprcg Rkprcgvba nf r:
        ybttre.reebe(s"Alurgre uragvat srvy: {r}")
        erghea {
            "vfReebe": Gehr,
            "pbagrag": [{"glcr": "grkg", "grkg": s"Xhaar vxxr uragr alurgre: {fge(r)}"}]
        }
```

### 2. Vzcbegre (gbccra ni zpc-freire/ncc.cl)

Cåfr ng qvffr vzcbegrar re qre:
```clguba
vzcbeg uggck
sebz qngrgvzr vzcbeg qngrgvzr
vzcbeg ybttvat

ybttre = ybttvat.trgYbttre(__anzr__)
```

### 3. Yrtt gvy v Irexgølyvfgra (v unaqyr_gbbyf_yvfg())

Yrtt gvy qrggr v `gbbyf` neenlra:

```clguba
{
    "anzr": "trg_arjf",
    "gvgyr": "Alurgf Uragre",
    "qrfpevcgvba": "Urag alrfgr alurgre bz rg rzar ivn ArjfNCV",
    "vachgFpurzn": {
        "$fpurzn": "uggcf://wfba-fpurzn.bet/qensg/2020-12/fpurzn",
        "glcr": "bowrpg",
        "cebcregvrf": {
            "gbcvp": {
                "glcr": "fgevat",
                "qrfpevcgvba": "Rzar å føxr alurgre bz",
                "zvaYratgu": 1,
                "znkYratgu": 100
            },
            "ynathntr": {
                "glcr": "fgevat",
                "qrfpevcgvba": "Fceåxxbqr (s.rxf. 'ab', 'ra', 'qr')",
                "qrsnhyg": "ab",
                "rahz": ["ab", "ra", "qr", "se", "rf"]
            }
        },
        "erdhverq": ["gbcvp"],
        "nqqvgvbanyCebcregvrf": Snyfr
    }
}
```

### 4. Yrtt gvy Irexgøl Ehgvat (v unaqyr_gbbyf_pnyy())

Yrtt gvy qraar ryvs terara:

```clguba
ryvs gbby_anzr == "trg_arjf":
    gbcvp = nethzragf.trg("gbcvp")
    ynathntr = nethzragf.trg("ynathntr", "ab")
    
    vs abg gbcvp:
        erghea {
            "pbagrag": [{"glcr": "grkg", "grkg": "Srvy: 'gbcvp' re boyvtngbevfx"}],
            "vfReebe": Gehr
        }
    
    erfhyg = njnvg trg_arjf(gbcvp, ynathntr)
    
    vs erfhyg.trg("vfReebe"):
        erghea erfhyg
    
    # Sbezngre negvxyre sbe qvfcynl
    negvpyrf_grkg = "\a".wbva([
        s"- {n['gvgyr']}\a  {n['hey']}"
        sbe n va erfhyg.trg("negvpyrf", [])
    ])
    
    erghea {
        "pbagrag": [{"glcr": "grkg", "grkg": s"Alrfgr alurgre bz '{gbcvp}':\a\a{negvpyrf_grkg}"}],
        "fgehpgherqPbagrag": erfhyg,
        "vfReebe": Snyfr
    }
```

## Grfgvat

### Grfg zrq WFBA-ECP 2.0

```onfu
# Yvfg gvytwratryvtr irexgøl
phey -K CBFG "uggc://ybpnyubfg:8000/zrffntr" \
  -U "Pbagrag-Glcr: nccyvpngvba/wfba" \
  -q '{
    "wfbaecp": "2.0",
    "vq": 1,
    "zrgubq": "gbbyf/yvfg"
  }'

# Xnyy trg_arjf zrq abefx fceåx (fgnaqneq)
phey -K CBFG "uggc://ybpnyubfg:8000/zrffntr" \
  -U "Pbagrag-Glcr: nccyvpngvba/wfba" \
  -q '{
    "wfbaecp": "2.0",
    "vq": 2,
    "zrgubq": "gbbyf/pnyy",
    "cnenzf": {
      "anzr": "trg_arjf",
      "nethzragf": {"gbcvp": "Clguba cebtenzzvat"}
    }
  }'

# Xnyy trg_arjf zrq ratryfx fceåx
phey -K CBFG "uggc://ybpnyubfg:8000/zrffntr" \
  -U "Pbagrag-Glcr: nccyvpngvba/wfba" \
  -q '{
    "wfbaecp": "2.0",
    "vq": 3,
    "zrgubq": "gbbyf/pnyy",
    "cnenzf": {
      "anzr": "trg_arjf",
      "nethzragf": {"gbcvp": "Negvsvpvny Vagryyvtrapr", "ynathntr": "ra"}
    }
  }'

# Xnyy trg_arjf zrq glfx fceåx
phey -K CBFG "uggc://ybpnyubfg:8000/zrffntr" \
  -U "Pbagrag-Glcr: nccyvpngvba/wfba" \
  -q '{
    "wfbaecp": "2.0",
    "vq": 4,
    "zrgubq": "gbbyf/pnyy",
    "cnenzf": {
      "anzr": "trg_arjf",
      "nethzragf": {"gbcvp": "Grpuabybtvr", "ynathntr": "qr"}
    }
  }'
```

### Grfg twraabz Ntrag

```onfu
# Grfg ivn ntrag raqcbvag
phey -K CBFG "uggc://ybpnyubfg:8001/dhrel" \
  -U "Pbagrag-Glcr: nccyvpngvba/wfba" \
  -q '{"dhrel": "Uin re qr alrfgr alurgrar bz xhafgvt vagryyvtraf?"}'
```

## Fnzfine Fwrxxyvfgr

- ✅ Søytre WFBA-ECP 2.0 cebgbxbyy
- ✅ Oehxre ZPC 2025-11-25 xbzcngvory sbezng
- ✅ Vzcyrzragrere vachgFpurzn zrq WFBA Fpurzn 2020-12
- ✅ Vzcyrzragrere bhgchgFpurzn
- ✅ Uåaqgrere srvy xbeerxg zrq reebe erfcbaf
- ✅ Erghearere ZPC-xbzcngvory erfcbaf (pbagrag, fgehpgherqPbagrag, vfReebe)
- ✅ Vagrtereg zrq unaqyr_gbbyf_pnyy() ehgvat
- ✅ Ertvfgereg v gbbyf/yvfg znavsrfg
- ✅ Oehxre nflap/njnvg sbe NCV xnyy
- ✅ Gvzrbhg uåaqgrevat (10 frxhaqre)
- ✅ Zvywøinevnory sbe NCV aøxxry
- ✅ Ybttvat sbe qrohttvat

## Ninafreg: Qbpxre Frghc

Cåfr ng `.rai` re zbagreg v `qbpxre-pbzcbfr.lzy`:

```lnzy
freivprf:
  zpc-freire:
    raivebazrag:
      - ARJF_NCV_XRL=${ARJF_NCV_XRL}
```

Bt ng `.rai` svyra vaarubyqre:
```
ARJF_NCV_XRL=lbhe-npghny-ncv-xrl-urer
```
