# Matriz de mapeo (producto | capa | componente | variable dashboard | función KQL propuesta)

| producto | capa | componente | variable dashboard | función KQL propuesta |
|---|---|---|---|---|
| MLP ADA | ingestas | dispatch | var_mlp_ada_dispatch | fn_mon_mlp_ada_ingestas("dispatch") |
| MLP ADA | ingestas | drillit | var_mlp_ada_drillit | fn_mon_mlp_ada_ingestas("drillit") |
| MLP ADA | ingestas | pi | var_mlp_ada_pi | fn_mon_mlp_ada_ingestas("pi") |
| MLP ADA | ingestas | plans | var_mlp_ada_plans | fn_mon_mlp_ada_ingestas("plans") |
| MLP ADA | ingestas | blockgrade | var_mlp_ada_blockgrade | fn_mon_mlp_ada_ingestas("blockgrade") |
| MLP ADA | ingestas | meteodata | var_mlp_ada_meteodata | fn_mon_mlp_ada_ingestas("meteodata") |
| MLP PDM CAEX | global | estado global | var_mlp_pdmcaex_global | fn_mon_mlp_pdmcaex_global() |
| MLP SIRO SAG | global | estado global | var_mlp_sirosag_global | fn_mon_mlp_sirosag_global() |
| CENTINELA | global | mtsulf | var_cent_mtsulf_status | fn_mon_cent_global("mtsulf") |
| CENTINELA | global | siro_mol | var_cent_siro_mol_status | fn_mon_cent_global("siro_mol") |
| CENTINELA | global | siro_flot | var_cent_siro_flot_status | fn_mon_cent_global("siro_flot") |
| CENTINELA | global | ada | var_cent_ada_status | fn_mon_cent_global("ada") |
| CENTINELA | global | mtox | var_cent_mtox_status | fn_mon_cent_global("mtox") |
| CENTINELA | global | mp10 | var_cent_mp10_status | fn_mon_cent_global("mp10") |
| CENTINELA | ingestas | dispatch | var_cent_dispatch | fn_mon_cent_ingestas("dispatch") |
| CENTINELA | ingestas | jigsaw | var_cent_jigsaw | fn_mon_cent_ingestas("jigsaw") |
| CENTINELA | ingestas | planes | var_cent_planes | fn_mon_cent_ingestas("planes") |
| CENTINELA | ingestas | vulcan | var_cent_vulcan | fn_mon_cent_ingestas("vulcan") |
| CENTINELA | ingestas | pi | var_cent_pi | fn_mon_cent_ingestas("pi") |
| AMSA CROSS PPFM | global | estado global | var_amsa_rep_ppfm_global | fn_mon_amsa_ppfm_global() |
| CMZ Reporte | transformacion | procesamiento | var_cmz_repEjecutivo_transformacion | fn_mon_cmz_rep_global("transformacion") |
