- This is GAIA WD /WD search roughly between eDRR3 WD sample and MS targeting sources in that area that behave well in GAIA DR3 and have W1-W2 > 0.3 in Catwise and are not contaminated in 5" from other Gaia sources.

- Disclaimer: this is very casual check to see if anything pops up. 

- Gaia search: 

query = """ 
SELECT *,
(phot_g_mean_mag + 5 * LOG10(parallax) - 10) AS GAIA_g_abs
FROM gaiadr3.gaia_source AS s
WHERE parallax > 2
  AND parallax_over_error > 4
  AND (phot_g_mean_mag + 5 * LOG10(parallax) - 10) > 7.5
  AND (phot_g_mean_mag + 5 * LOG10(parallax) - 10) > 2.5 * (phot_bp_mean_mag - phot_rp_mean_mag) + 5
  AND (phot_g_mean_mag + 5 * LOG10(parallax) - 10) < 6 + (5 * (bp_rp))
"""

Filters: 

filtered_df = df[
    (df['parallax_over_error'] > 4) &
    (df['visibility_periods_used'] >= 6) &
    (df['phot_g_mean_flux_over_error'] > 40) &
    (df['phot_bp_n_obs'] >= 6) &
    (df['phot_bp_mean_flux_over_error'] > 4) &
    (df['phot_rp_n_obs'] >= 6) &
    (df['phot_rp_mean_flux_over_error'] > 4) &
    (df['astrometric_excess_noise'] < 1) &
    (df['in_qso_candidates'] == False) &
    (df['in_galaxy_candidates'] == False) &
    (df['classprob_dsc_combmod_star'] > 0.9)

]

phot_bp_rp_excess_ok = (
    df['phot_bp_rp_excess_factor'] > 1.0 + 0.015 * (bp_rp ** 2)
) & (
    df['phot_bp_rp_excess_factor'] < 1.3 + 0.06 * (bp_rp ** 2)
)

filtered_df = filtered_df[phot_bp_rp_excess_ok]


Made generous room to avoid contamination:
SELECT 
    A.*
FROM 
    gaia_result AS A
LEFT OUTER JOIN gaiadr2.gaia_source AS B
    ON 1 = CONTAINS(
        POINT('ICRS', A.ra, A.dec),
        CIRCLE('ICRS', B.ra, B.dec, 5./3600)
    )
    AND A.SOURCE_ID != B.source_id
WHERE 
    B.source_id IS NULL


Crossmatched with Catwise2020 using W1 and W2 filters as follows:
(ab_flags = '00' AND abs(glat) > 10 AND( (w2mpro BETWEEN 13 AND 16 AND ((w1mpro - w2mpro) > 0.3 OR (w2snr / w1snr > 3 AND w2mpro > 15.6))) OR (w1mpro IS NULL AND w2mpro > 15) ) AND cc_flags NOT LIKE '_D__' AND cc_flags NOT LIKE '_H__' AND cc_flags NOT LIKE '_O__' AND cc_flags NOT LIKE '_P__')
