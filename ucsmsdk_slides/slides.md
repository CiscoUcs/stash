# Cisco Ucs Python SDK #
---
# Installation
    pip install ucsmsdk

---
# Agenda #
* Introducing the changes
    * Structural changes
    * Consistency changes
    * Format changes
    * Optimization changes
* Sample Scripts
* Convert-to-python
    * Demo

---
# Structural changes #
# top level #
    $ls ucsmsdk
    __init__.py            ucsfilter.py
    methodmeta             ucsfiltertype.py
    mometa                 ucsgenutils.py
    ucsbasetype.py         ucshandle.py
    ucsconfig.cfg          ucsmeta.py
    ucsconnectiondriver.py ucsmethod.py
    ucsconstants.py        ucsmethodfactory.py
    ucscore.py             ucsmo.py
    ucscoremeta.py         ucssession.py
    ucscoreutils.py        ucsxmlcodec.py
    ucseventhandler.py     utils
    ucsexception.py
    $
---
# Structural changes #
# mo components #
    $ls ucsmsdk/mometa
    __init__.py domain      fsm         lstorage    qosclass
    aaa         dpsec       gmeta       macpool     queryresult
    adaptor     dupe        graphics    memory      security
    ape         epqos       hostimg     mgmt        sol
    bios        equipment   ident       network     stats
    bmc         ether       imgprov     nfs         storage
    callhome    event       imgsec      nwctrl      sw
    capability  extmgmt     initiator   observe     swat
    change      extpol      ip          org         synthetic
    cimcvmedia  extvmm      ippool      os          sysdebug
    clitest     fabric      iqnpool     pci         sysfile
    comm        fault       iscsi       pki         top
    compute     fc          license     policy      trig
    config      fcpool      lldp        port        uuidpool
    dcx         feature     ls          power       version
    dhcp        firmware    lsboot      proc        vm
    diag        flowctrl    lsmaint     processor   vnic
    $
---
# Structural changes #
# mo and mo_meta #
    $ls ucsmsdk/mometa/ls
    LsAgentPolicy.py         LsServerExtension.py
    LsBinding.py             LsServerFsm.py
    LsFcLocale.py            LsServerFsmStage.py
    LsFcZone.py              LsServerFsmTask.py
    LsFcZoneGroup.py         LsTier.py
    LsIdentityInfo.py        LsUuidHistory.py
    LsIssues.py              LsVConAssign.py
    LsPower.py               LsVersionBeh.py
    LsRequirement.py         LsZoneInitiatorMember.py
    LsServer.py              LsZoneTargetMember.py
    LsServerAssocCtx.py      __init__.py
    $
---
# Consistency changes #
    * PEP8 compliance
    * Pylint/PyFlakes compliance
    * file naming - most files have a "ucs" prefix now
    * function naming - is made more consistent
    * code separation - logical separation of code
---
# Session
    !python
    from ucsmsdk.ucshandle import UcsHandle

    # init handle
    handle = UcsHandle("10.104.223.192", "admin", "password")

    # login
    handle.login()

    # print session cookie
    print handle.cookie

    #logout
    handle.logout()


# Output
    1446901277/e9302814-792c-4eb6-8ed2-661505b133bb
---
# query_dn
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    mo = handle.query_dn("org-root")
    print mo

    handle.logout()

# Output
    Managed Object          :   OrgOrg
    --------------
    child_action                    :None
    descr                           :
    dn                              :org-root
    flt_aggr                        :65536
    level                           :root
    name                            :root
    perm_access                     :yes
    rn                              :org-root
    sacl                            :None
    status                          :None

---
# query_dns
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    mo_dict = handle.query_dns("org-root", "fabric/lan/net-default")
    print mo_dict

    handle.logout()

# Output
    {
     'org-root': 
        <ucsmsdk.mometa.org.OrgOrg.OrgOrg object at 0x109dc5f50>, 
     'fabric/lan/net-default': 
        <ucsmsdk.mometa.fabric.FabricVlan.FabricVlan object at 0x109dc5ed0>
    }
---
# query_classid
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    mo_array = handle.query_classid("orgOrg")
    print mo_array

    handle.logout()

# Output
    [<ucsmsdk.mometa.org.OrgOrg.OrgOrg object at 0x109dc5890>, 
     <ucsmsdk.mometa.org.OrgOrg.OrgOrg object at 0x109dc5810>]
    
---
# query_classids
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    mo_dict = handle.query_classids("orgOrg", "fabricVlan")
    print mo_dict

    handle.logout()

# Output
    {
     'OrgOrg': 
        [<ucsmsdk.mometa.org.OrgOrg.OrgOrg object at 0x109de0150>, 
         <ucsmsdk.mometa.org.OrgOrg.OrgOrg object at 0x109de00d0>],
     'FabricVlan':
        [<ucsmsdk.mometa.fabric.FabricVlan.FabricVlan object at 0x109de0090>,
         <ucsmsdk.mometa.fabric.FabricVlan.FabricVlan object at 0x109de0110>]
     }
---
# add_mo
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    # import Mo class
    # create the client Mo
    # add Mo to session handler
    # commit the changes

    from ucsmsdk.mometa.ls.LsServer import LsServer
    sp = LsServer(parent_mo_or_dn="org-root", name="sp_demo")
    handle.add_mo(sp)
    handle.commit()

    handle.logout()
---
# set_mo
    !python
    from ucsmsdk.ucshandle import UcsHandle
    handle = UcsHandle("10.104.223.192", "admin", "password")
    handle.login()

    # query the object 
    # change a property
    # set_mo
    # commit

    sp = handle.query_dn("org-root/ls-sp_demo")
    sp.descr = "demo_descr"
    handle.set_mo(sp)
    handle.commit()

    handle.logout()
---
# filter 
# string based filter support
    !python
    filter = '(prop1, value1)'
    filter = '(prop1, value1) or (prop2, value2)'
    filter = '''(prop1, value1, type="eq") and 
                ((prop2, value2) or (prop2, value3))'''
    filter = '''(prop1, value1, type="re", flag="I") and 
                ((prop2, value2) or (prop2, value3))'''
---
# filter - sample data
    !python
    sps = []
    from ucsmsdk.mometa.ls.LsServer import LsServer
    sps.append(LsServer(parent_mo_or_dn="org-root", name="demo_1"))
    sps.append(LsServer(parent_mo_or_dn="org-root", name="demo_1_1"))
    sps.append(LsServer(parent_mo_or_dn="org-root", name="demo_1_2"))
    sps.append(LsServer(parent_mo_or_dn="org-root", name="demo_2_1"))
    sps.append(LsServer(parent_mo_or_dn="org-root", name="demo_2_2"))
    sps.append(LsServer(parent_mo_or_dn="org-root", name="DEMO"))
    for sp in sps:
        handle.add_mo(sp)
    handle.commit()

    sp = handle.query_classid(class_id="LsServer")
    for mo in sp:
        print mo.name,
# Output
    DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
---
# filter
# default match is regex
    !python
    filter_str = '(name,"demo_1")'
    
    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,

# Output
    Filter      : (name,"demo_1")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : demo_1 demo_1_1 demo_1_2
---
# filter
    !python
    filter_str = '(name,"demo_1") or (name, "demo_2")'

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,

# Output
    Filter      : (name,"demo_1") or (name, "demo_2")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2    
---
# filter
    !python
    filter_str = '(name,"[0-9]_1")'

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,

# Output
    Filter      : (name,"[0-9]_1")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : demo_1_1 demo_2_1
---
# filter
    !python
    filter_str = '(name,"demo_1",type="eq")'

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,


# Output
    Filter      : (name,"demo_1",type="eq")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : demo_1
---
# filter
    !python
    filter_str = '(name,"DEMO",type="re", flag="I")'

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,

# Output
    Filter      : (name,"DEMO",type="re", flag="I")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
---
# filter
    !python
    filter_str = '(name,"DEMO",type="re")'

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,
# Output
    Filter      : (name,"DEMO",type="re")
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : DEMO
---
# filter
    !python
    filter_str = '''(name,"demo",type="re") and 
                        ((name,"demo_1") and (name, "[0-9]_1"))'''

    sp = handle.query_classid(class_id="LsServer", filter_str=filter_str)
    for mo in sp:
        print mo.name,
# Output
    Filter      : (name,"demo",type="re") and 
                        ((name,"demo_1") and (name, "[0-9]_1"))
    All         : DEMO demo_1 demo_1_1 demo_2_1 demo_1_2 demo_2_2
    Filtered    : demo_1_1
---
# property validation at client
    !python
    from ucsmsdk.mometa.ls.LsServer import LsServer
    try:
        sp = LsServer(parent_mo_or_dn="org-root", 
                        name="sp_demo_really_really_really_long")
    except Exception as e:
        print e
# output
    DEBUG:ucs:nameshould adhere to regex^[\-\.:_a-zA-Z0-9]{2,32}$
    Invalid Value Exception - [LsServer]: Prop <name>, 
                Value<sp_demo_really_really_really_long>.
---
# property validation at client
    !python
    from ucsmsdk.mometa.ls.LsVConAssign import LsVConAssign
    try:
        mo = LsVConAssign(parent_mo_or_dn=sp, admin_vcon="any", order="1",
                        transport="would_produce_error", vnic_name="eth0")
    except Exception as e:
        print e
# output
    DEBUG:ucs:transportshould adhere to 
        regex^((fc|ethernet|defaultValue),){0,2}(fc|ethernet|defaultValue){0,1}$
    Invalid Value Exception - [LsVConAssign]: 
        Prop <transport>, Value<would_produce_error>. 
---
# real world use case #
    !python
    create                          SP
    associate                       SP
    wait_for_assoc                  SP
    association_done                SP
    deassociate                     SP
    wait_for_deassoc                SP
    deassociation_done              SP
    wait_for_blade_availability     Blade
    blade_available                 Blade
    delete                          SP    
---
# sp_create #
    !python
    def sp_create():
        # ###########################################
        # Create the SP that will be associated later
        # ###########################################
        log.debug("sp_create")
        from ucsmsdk.mometa.ls.LsServer import LsServer
        from ucsmsdk.mometa.lsboot.LsbootDef import LsbootDef
        from ucsmsdk.mometa.lsboot.LsbootStorage import LsbootStorage
        from ucsmsdk.mometa.lsboot.LsbootLocalStorage import LsbootLocalStorage
        from ucsmsdk.mometa.lsboot.LsbootDefaultLocalImage import\
            LsbootDefaultLocalImage
        from ucsmsdk.mometa.vnic.VnicEther import VnicEther
        from ucsmsdk.mometa.vnic.VnicEtherIf import VnicEtherIf
        from ucsmsdk.mometa.vnic.VnicFcNode import VnicFcNode

        mo = LsServer(parent_mo_or_dn="org-root", vmedia_policy_name="",
                      ext_ip_state="none", bios_profile_name="",
                      mgmt_fw_policy_name="", agent_policy_name="",
                      mgmt_access_policy_name="", dynamic_con_policy_name="",
                      kvm_mgmt_policy_name="", sol_policy_name="", uuid="0",
                      descr="", stats_policy_name="default", policy_owner="local",
                      ext_ip_pool_name="ext-mgmt", boot_policy_name="", usr_lbl="",
                      host_fw_policy_name="", vcon_profile_name="",
                      ident_pool_name="", src_templ_name="",
                      local_disk_policy_name="", scrub_policy_name="",
                      power_policy_name="default", maint_policy_name="",
                      name="test_sp", resolve_remote="yes")
        mo_1 = LsbootDef(parent_mo_or_dn=mo, descr="", reboot_on_update="no",
                         adv_boot_order_applicable="no", policy_owner="local",
                         enforce_vnic_name="yes", boot_mode="legacy")
        mo_1_1 = LsbootStorage(parent_mo_or_dn=mo_1, order="1")
        mo_1_1_1 = LsbootLocalStorage(parent_mo_or_dn=mo_1_1, )
        mo_1_1_1_1 = LsbootDefaultLocalImage(parent_mo_or_dn=mo_1_1_1, order="1")
        mo_2 = VnicEther(parent_mo_or_dn=mo, nw_ctrl_policy_name="", name="eth0",
                         admin_host_port="ANY", admin_vcon="any",
                         stats_policy_name="default", admin_cdn_name="",
                         switch_id="A", pin_to_group_name="", mtu="1500",
                         qos_policy_name="", adaptor_profile_name="",
                         ident_pool_name="", order="unspecified", nw_templ_name="",
                         addr="derived")
        mo_2_1 = VnicEtherIf(parent_mo_or_dn=mo_2, default_net="yes",
                             name="default")
        mo_3 = VnicFcNode(parent_mo_or_dn=mo, ident_pool_name="",
                          addr="pool-derived")
        handle.add_mo(mo)
        handle.commit()
---
# sp_assoc #
    !python
    def sp_assoc():
        # ###########################################
        # associate the SP to a blade
        # ###########################################
        log.debug("sp_assoc")
        from ucsmsdk.mometa.ls.LsBinding import LsBinding
        mo = LsBinding(parent_mo_or_dn="org-root/ls-test_sp",
                       pn_dn=blade_dn, restrict_migration="no")
        handle.add_mo(mo)
        handle.commit()

        # ###########################################
        # add a handler to watch for sp assoc
        # ###########################################
        mo = handle.query_dn("org-root/ls-test_sp")
        sp_assoc_wait(mo)
---
# sp_assoc_wait #
    !python
    def sp_assoc_wait(mo):
        # ###########################################
        # Create a Event handler to monitor sp assoc
        # ###########################################
        log.debug("sp_assoc_wait")
        ucs_event_handler.add(
                    managed_object=mo,
                    prop="assoc_state",
                    success_value=[LsServerConsts.ASSOC_STATE_ASSOCIATED],
                    failure_value=[LsServerConsts.ASSOC_STATE_FAILED],
                    timeout_sec=600,
                    call_back=sp_assoc_wait_done_cb)
---
# sp_assoc_wait_done_cb #
    !python
    def sp_assoc_wait_done_cb(mce):
        log.debug("sp_assoc_wait_done_cb")
        log.debug("SP:" + mce.mo.dn + " Assoc Done Callback. assoc_state: " +
                  mce.mo.assoc_state)
        if mce.mo.assoc_state == LsServerConsts.ASSOC_STATE_ASSOCIATED:
            sp_deassoc()
---
# sp_deassoc #
    !python
    def sp_deassoc():
        # ###########################################
        # add a handler to watch for sp assoc
        # ###########################################
        mo = handle.query_dn("org-root/ls-test_sp")
        sp_deassoc_wait(mo)

        # ###########################################
        # deassociate the SP
        # ###########################################
        log.debug("sp_deassoc")
        obj = handle.query_dn("org-root/ls-test_sp/pn")
        handle.remove_mo(obj)
        handle.commit()
---
# sp_deassoc_wait #
    !python
    def sp_deassoc_wait(mo):
        # ###########################################
        # Create a Event handler to monitor sp assoc
        # ###########################################
        log.debug("sp_deassoc_wait")
        ucs_event_handler.add(
                    managed_object=mo,
                    prop="assoc_state",
                    success_value=[LsServerConsts.ASSOC_STATE_UNASSOCIATED],
                    failure_value=[LsServerConsts.ASSOC_STATE_FAILED],
                    timeout_sec=600,
                    call_back=sp_deassoc_wait_done_cb)
---
# sp_deassoc_wait_done_cb #
    !python
    def sp_deassoc_wait_done_cb(mce):
        log.debug("sp_deassoc_wait_done_cb")
        log.debug("SP:" + mce.mo.dn + " Deassoc Done Cb. assoc_state: " +
                  mce.mo.assoc_state)
        if mce.mo.assoc_state == LsServerConsts.ASSOC_STATE_UNASSOCIATED:
            blade_wait_for_availability()
---
# blade_wait_for_availability #
    !python
    def blade_wait_for_availability():
        log.debug("blade_wait_for_availability")
        mo = handle.query_dn("sys/chassis-1/blade-1")
        ucs_event_handler.add(
                    managed_object=mo,
                    prop="availability",
                    success_value=['available'],
                    timeout_sec=600,
                    poll_sec=10,
                    call_back=blade_wait_for_availability_done_cb)
---
# blade_wait_for_availability_done_cb #
    !python
    def blade_wait_for_availability_done_cb(mce):
        log.debug("blade_wait_for_availability_done_cb")
        if mce.mo.availability == 'available':
            sp_delete()
        handle.logout()
---
# sp_assoc_deassoc with event handlers
    !python
    def main():
        sp_create()
        sp_assoc()

# output
    !python
    ucs - DEBUG - sp_create
    ucs - DEBUG - sp_assoc
    ucs - DEBUG - sp_assoc_wait
    ucs - DEBUG - sp_assoc_wait_done_cb
    ucs - DEBUG - SP:org-root/ls-test_sp Assoc Done Callback. 
                                    assoc_state: associated
    ucs - DEBUG - sp_deassoc_wait
    ucs - DEBUG - sp_deassoc
    ucs - DEBUG - sp_deassoc_wait_done_cb
    ucs - DEBUG - SP:org-root/ls-test_sp Deassoc Done Callback. 
                                    assoc_state: unassociated
    ucs - DEBUG - blade_wait_for_availability
    ucs - DEBUG - blade_wait_for_availability_done_cb
    ucs - DEBUG - sp_delete
---
# policy #
    !python
    # #####################################
    # Create a Flow Control Policy
    # #####################################
    from ucsmsdk.mometa.flowctrl.FlowctrlItem import FlowctrlItem
    mo = FlowctrlItem(parent_mo_or_dn="fabric/lan/flowctrl", snd="off",
                      rcv="off", name="test", prio="on")


    # #####################################
    # Create Dynamic vNIC Connection Policy
    # #####################################
    from ucsmsdk.mometa.vnic.VnicDynamicConPolicy import \
                                        VnicDynamicConPolicy
    mo = VnicDynamicConPolicy(parent_mo_or_dn="org-root", name="test",
                              descr="test", adaptor_profile_name="Linux",
                              policy_owner="local",
                              protection="protected-pref-b", 
                              dynamic_eth="54")
---
# policy #
    !python
    # #####################################
    # Create LACP Policy
    # #####################################
    from ucsmsdk.mometa.fabric.FabricLacpPolicy import FabricLacpPolicy
    mo = FabricLacpPolicy(parent_mo_or_dn="org-root", fast_timer="fast",
                          policy_owner="local", suspend_individual="true",
                          name="test", descr="")

    # #####################################
    # Modify LACP Policy
    # #####################################
    obj = handle.query_dn("org-root/lacp-test")
    obj.fast_timer = "normal"
    obj.suspend_individual = "false"
    obj.policy_owner = "local"
    obj.descr = ""
    handle.set_mo(obj)
---
# policy #
    !python
    # #####################################
    # Create LAN Connectivity Policy
    # #####################################
    from ucsmsdk.mometa.vnic.VnicLanConnPolicy import VnicLanConnPolicy
    from ucsmsdk.mometa.vnic.VnicEther import VnicEther
    from ucsmsdk.mometa.vnic.VnicEtherIf import VnicEtherIf

    mo = VnicLanConnPolicy(parent_mo_or_dn="org-root", policy_owner="local",
                           name="test", descr="test_policy")
    mo_1 = VnicEther(parent_mo_or_dn=mo, nw_ctrl_policy_name="default",
                     name="test", admin_host_port="ANY", admin_vcon="any",
                     stats_policy_name="default", admin_cdn_name="",
                     switch_id="A", pin_to_group_name="", mtu="1500",
                     qos_policy_name="qos-1", adaptor_profile_name="Linux",
                     ident_pool_name="mac-pool-1", order="1", 
                     nw_templ_name="", addr="derived")
    mo_1_1 = VnicEtherIf(parent_mo_or_dn=mo_1, default_net="yes",
                         name="default")
---
# policy #
    !python 
    # #####################################
    # Create QoS Policy
    # #####################################
    from ucsmsdk.mometa.epqos.EpqosDefinition import EpqosDefinition
    from ucsmsdk.mometa.epqos.EpqosEgress import EpqosEgress

    mo = EpqosDefinition(parent_mo_or_dn="org-root", policy_owner="local",
                         name="test", descr="")
    mo_1 = EpqosEgress(parent_mo_or_dn=mo, rate="line-rate",
                       host_control="none", name="", prio="best-effort",
                       burst="10240")
---
# convert_to_ucs_python #
* GUI considers a few XML snippets as secure and does not log them. So, the convert_to_ucs_python does not find the logs to do the translation. The workaround is as follows:
    * Save the launch link as a file … i.e. ``` https://<ip_or_hostname>/ucsm/ucsm.jnlp ```
    * Right click on the file, Open with notepad/vim, and add the following line after the other property definitions.

        ```xml 
           <property name="jnlp.ucsm.log.show.encrypted" value="true"/>
        ```
    * Right click on the file,  Open with “Java™ Web Start Launcher”
    * After the UI launches, from a Python Shell run the convert_to_ucs_python command

---
# convert_to_ucs_python #
* On some operating systems, the program might encounter issues discovering the path to which UI logs are written.
* convert_to_ucs_python does not work if log path is not detected.
* Follow the below steps, if case of log_path issues,
    ```code
        $cd ~
        $sudo find . -name '.ucsm'
        ./Library/Application Support/Oracle/Java/Deployment/log/.ucsm
    ```
* so now you can specify log_path argument to convert_to_ucs_python to "./Library/Application Support/Oracle/Java/Deployment/log/.ucsm"

---
# Thank you #
---
# vvb@cisco.com #
