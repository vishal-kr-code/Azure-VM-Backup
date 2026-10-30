# Azure-VM-Backup
Azure VM Backup

#Connect-AzAccount -Environment AzureChinaCloud -UseDeviceAuthentication
#Connect-AzAccount
Connect-AzAccount -DeviceCode
# Azure Backup Monthly Report - October 2026
# Azure Backup stores and queries job times in UTC.

$Ranges = @(
    @{
        Start = [DateTime]::SpecifyKind(
            (Get-Date "2026-10-01T00:00:00"), 'Utc'
        )
        End = [DateTime]::SpecifyKind(
            (Get-Date "2026-10-15T23:59:59"), 'Utc'
        )
    },
    @{
        Start = [DateTime]::SpecifyKind(
            (Get-Date "2026-10-16T00:00:00"), 'Utc'
        )
        End = [DateTime]::SpecifyKind(
            (Get-Date "2026-10-30T23:59:59"), 'Utc'
        )
    }
)

$Subscriptions = Get-AzSubscription
$AllReports = @()

foreach ($Subscription in $Subscriptions) {

    if ($Subscription.State -eq "Enabled") {

        Write-Output "Processing Subscription: $($Subscription.Name)"

        Select-AzSubscription -SubscriptionId $Subscription.Id | Out-Null

        $Vaults = Get-AzRecoveryServicesVault

        foreach ($Vault in $Vaults) {

            Write-Output "Processing Vault: $($Vault.Name)"

            Set-AzRecoveryServicesVaultContext -Vault $Vault

            foreach ($Range in $Ranges) {

                # Fetch Azure VM Backup jobs (UTC-based query)

                $Backup_Jobs = Get-AzRecoveryServicesBackupJob `
                    -Operation Backup `
                    -BackupManagementType AzureVM `
                    -From $Range.Start `
                    -To $Range.End

                foreach ($Job in $Backup_Jobs) {

                    $AllReports += [PSCustomObject]@{
                        WorkloadName  = $Job.WorkloadName
                        Operation     = $Job.Operation
                        VaultName     = $Vault.Name
                        Status        = $Job.Status
                        StartTime_UTC = $Job.StartTime
                        EndTime_UTC   = $Job.EndTime
                        Duration      = $Job.Duration
                        Subscription  = $Subscription.Name
                    }
                }
            }
        }
    }
}

# Export consolidated monthly report

$ReportPath = "C:\Users\OneDriveInc\New Document\Oct2026_Report.csv"

$AllReports |
    Export-Csv -Path $ReportPath -NoTypeInformation

Write-Output "Successfully merged $($AllReports.Count) jobs from October 2026."

Write-Output "Report saved at: $ReportPath"
