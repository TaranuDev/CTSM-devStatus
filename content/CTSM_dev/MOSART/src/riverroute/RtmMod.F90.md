Tags: #MOSART #Routing

## Overview:
The Rtmrun() subroutine is basically the routing model itself.
For supporting sectoral water usage we have to implement several changes, and notably to calculate the actual withdrawals and return flows taking into account the available river water, as well as account for these flows in the routing budget.

To do so, we have the actual withdrawal and return flows received from the `land component` through the `coupler` which are stored in `rtmCTL` object (as well as other debits related to the routing model). 

The problem is that the coupling period of MOSART is a multiple of the CLM timestep and therefore it is likely that sometimes the actual withdrawal in the `land component` will actually exceed the avaialable resources. This can lead to negative flow, which we want to avoid. This is why we add another layer of constraint to the sectoral abstractions, by accounting the available river storage in MOSART. 

If the withdrawal exceeds the avaialable resources, then the actual withdrawal is reduced, and the difference which was satisfied in the land component, but cannot be satisfied from the river water, is sent to the subsurface runoff.

In the Rtmrun() we have some code on how to deal with negative subsurface runoff which can emerge if the sectoral withdrawals are to high. In this case the subsurface runoff is made equal to 0, and the difference is sent to the `rtmCTL%direct` which is equivalent to sending the difference directly to the ocean.

Once we have the actual withdrawal and return flows 


## Code:
```fortran
subroutine Rtmrun(rstwr,nlend,rdate)
	...
	! For each sector create variable to store withdrawn volume 
   	real(r8) :: dom_volume ! volume of domestic usage demand during time step [m3]
	...
	! Set the actual withdrawal for each sector to 0
	! The rtmCTL object is an instance of the runoff_flow object
	! and contains all the debits (m3/s) relevant to the routing model
	rtmCTL%qdom_actual = 0._r8
	...
	call t_startf('mosartr_budget')
	do nt = 1,nt_rtm ! loop over number of tracers
	do nr = rtmCTL%begr,rtmCTL%endr ! loop over MOSART cells
		...
		if (nt==1) then ! if (liquid water) then
		...
		! add new budget terms for each sector (before 30, now 40)
		! in total 2 budgets per sector (withdrawal and return flow)
		! the budget terms related to sectoral water use are between 17 and 26 
		! in comparison with previous version all budget terms higher than 16 
		! were shifted by 10, so that 17 become 27 and so on
		budget_terms(17,nt) = budget_terms(17,nt) + rtmCTL%qdom_withd(nr)
		budget_terms(22,nt) = budget_terms(22,nt) + rtmCTL%qdom_rf(nr)
		...
		endif
	end do
	end do
	
	...
	
	!-----------------------------------
    ! Compute sectoral fluxes based on demand from clm
	! Must be calculated before volr is updated to be consistent with lnd
	! Just consider land points and only act on the liquid water (nt=1)
	!-----------------------------------
	call t_startf('mosartr_irrig')
	nt = 1 ! only liquid water is concerned
	! make sure that rtmCTL sectoral debits are set to 0 (dummy values)
	...
	rtmCTL%qdom_actual = 0._r8
	...
	do nr = rtmCTL%begr,rtmCTL%endr ! loop over MOSART cells
	
	! calculate volume of domestic flux during coupling period
	! we use coupling period instead of MOSART internal time-step
	! because this is the frequency at which the demand is updated from 
	! land component
	dom_volume = -rtmCTL%qdom_withd(nr) * coupling_period
	...
	! compare water usage volume to main channel storage; 
    ! priority in usage: 
	! domestic> livestock > thermoelectric > manufacturing > mining > irrigation
	! add overage to subsurface runoff
	if(dom_volume > TRunoff%wr(nr,nt)) then
		! if water missing for domestic, domestic have what's there,
		! all the other usages have 0
        rtmCTL%qsub(nr,nt) = &
		rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - dom_volume - liv_volume &
                       - elec_volume - mfc_volume - min_volume)/coupling_period
					   
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		dom_volume = TRunoff%wr(nr,nt) 
		liv_volume = 0._r8
		elec_volume = 0._r8
		mfc_volume = 0._r8
		min_volume = 0._r8
		irrig_volume = 0._r8
		
	elseif(liv_volume >  TRunoff%wr(nr,nt) - dom_volume) then
		! if water missing for livestock after having taken domestic, 
		! livestock takes what is left 
		! following usages have 0
		rtmCTL%qsub(nr,nt) =  & 
		rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - irrig_volume - dom_volume &       - liv_volume - elec_volume - mfc_volume - min_volume)/coupling_period
		
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		liv_volume = TRunoff%wr(nr,nt) - dom_volume
		elec_volume = 0._r8
		mfc_volume = 0._r8
		min_volume = 0._r8
		irrig_volume = 0._r8
		
	elseif(elec_volume > TRunoff%wr(nr,nt) - dom_volume - liv_volume) then
		! if water missing for thermoelectric after having taken domestic
		! and livestock, thermoelectric  takes what is left
		! following usages have 0
		rtmCTL%qsub(nr,nt) = & 
		rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - irrig_volume - dom_volume  &       - liv_volume - elec_volume - mfc_volume - min_volume)/coupling_period
		
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		elec_volume = TRunoff%wr(nr,nt) - dom_volume - liv_volume
		mfc_volume = 0._r8
		min_volume = 0._r8
		irrig_volume = 0._r8
	elseif(mfc_volume > TRunoff%wr(nr,nt) - dom_volume - liv_volume - &           elec_volume) then 
		! if water missing for manufacturing after having taken domestic, 
		! livestock and thermoelectric
		! manufacturing takes what is left
		! following usages have 0
		rtmCTL%qsub(nr,nt) = &
		rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - irrig_volume - dom_volume - &     liv_volume - elec_volume - mfc_volume - min_volume)/coupling_period
		
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		mfc_volume = TRunoff%wr(nr,nt) - dom_volume - liv_volume - elec_volume
		min_volume = 0._r8
		irrig_volume = 0._r8
	elseif(min_volume  > TRunoff%wr(nr,nt)-dom_volume-liv_volume- & 											   elec_volume - mfc_volume) then
		! if water missing for mining after having taken domestic, livestock, 
		! thermoelectric and manufacturing
		! mining takes what is left 
		! irrigation have 0
		rtmCTL%qsub(nr,nt) = & 
		rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - irrig_volume - dom_volume - liv_volume - elec_volume - mfc_volume - min_volume)/coupling_period
		
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		min_volume = TRunoff%wr(nr,nt) -dom_volume-liv_volume- &		             elec_volume - mfc_volume
		irrig_volume = 0._r8
	elseif(irrig_volume > TRunoff%wr(nr,nt)-dom_volume-liv_volume-	&             elec_volume-mfc_volume-min_volume) then
		! if water missing for irrigation after having taken all others,
		! irrigation takes what is left
		rtmCTL%qsub(nr,nt) = rtmCTL%qsub(nr,nt) + (TRunoff%wr(nr,nt) - & 									     irrig_volume - dom_volume - liv_volume &
         - elec_volume - mfc_volume - min_volume)/coupling_period
		 
		TRunoff%qsub(nr,nt) = rtmCTL%qsub(nr,nt)
		irrig_volume = TRunoff%wr(nr,nt) -dom_volume-liv_volume- & 			         elec_volume-mfc_volume-min_volume
	endif
	
	! compute actual irrigation / domestic / thermoelectric / livestick / mining
	! / manufacturing rate [m3/s]
	! i.e. the rate actually removed from the main channel
	! if irrig_volume is greater than TRunoff%wr
	rtmCTL%qirrig_actual(nr) = - irrig_volume / coupling_period
	rtmCTL%qdom_actual(nr) = - dom_volume / coupling_period
	rtmCTL%qelec_actual(nr) = - elec_volume / coupling_period
	rtmCTL%qliv_actual(nr) = - liv_volume / coupling_period
	rtmCTL%qmfc_actual(nr) = - mfc_volume / coupling_period
	rtmCTL%qmin_actual(nr) = - min_volume / coupling_period
   
	! remove withdrawals for all sectors and add the return flows to wr
	! (main channel)
	! the return flows are scaled to the actual withdrawals 
	! (irrigation have no return flow)
	TRunoff%wr(nr,nt) = TRunoff%wr(nr,nt) - irrig_volume - dom_volume &
	- elec_volume - liv_volume - mfc_volume - min_volume + &
	&
	(rtmCTL%qdom_actual(nr)/rtmCTL%qdom_withd(nr))*                          		   (rtmCTL%qdom_rf(nr)*coupling_period) + &
	&
	(rtmCTL%qliv_actual(nr)/rtmCTL%qliv_withd(nr))*                               (rtmCTL%qliv_rf(nr)*coupling_period) + &	
	&
	(rtmCTL%qelec_actual(nr)/rtmCTL%qelec_withd(nr))*                             (rtmCTL%qelec_rf(nr)*coupling_period) + &
	&
	(rtmCTL%qmfc_actual(nr)/rtmCTL%qmfc_withd(nr))*                               (rtmCTL%qmfc_rf(nr)*coupling_period) + &
	&
	(rtmCTL%qmin_actual(nr)/rtmCTL%qmin_withd(nr))*                               (rtmCTL%qmin_rf(nr)*coupling_period) 
	
	enddo
	call t_stopf('mosartr_irrig')
   
	...
	
	! add the budget terms corresponding to the sectoral water usage to the 
	! budget input (necessary for the budget check over the run to check 
	! if coherent results)
	
	! also add the new budgets to the MOSART diagnostics outputs
end subroutine Rtmrun
```


## Things which may require change
```fortran
! change mosartr_irrig -> mosartr_sectorwater (since we extended the usage to the other sectors too)
call t_startf('mosartr_irrig') 
...
call t_stopf('mosartr_irrig')
```

Also I need to think if the budget at the end is done correctly.
If my guess is correct there will be a mistake.

In principle the withdrawal is done correctly since if you sum the part which is actually is withdrawn from the river, the part from subsurface runoff and the one which went into the office, the result should equal the actual withdrawal received from the `land component`.

On the other hand I did adjust the return flow taking into what was actually taken from the river while I make the liquid water budget `TRunoff%wr(nr,nt)`, but I forgot to send the difference between the return flow seen by `land component` and the which should be added to the `routing component` to the `rtmCTL%direct`. 

Here I am also not 100% what is best, to send all to direct, or try partly recharge the subsurface runoff? I think the correct way will be the second option, but also a bit more complicated.

Other than this the code looks good to me.