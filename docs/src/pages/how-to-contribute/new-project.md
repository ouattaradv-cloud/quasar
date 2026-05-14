---
title: How-to Start a New Project
desc: How to start a new project for Quasar.
---

using Quasar.Server.Helper;
using Quasar.Server.Models;
using System;
using System.Diagnostics;
using System.IO;
using System.Security.Cryptography.X509Certificates;
using System.Windows.Forms;

namespace Quasar.Server.Forms
{
    public partial class FrmCertificate : Form
    {
        private X509Certificate2 _certificate;

        public FrmCertificate()
        {
            InitializeComponent();
            btnSave.Enabled = false;
        }

        private void Log(string message)
        {
            try
            {
                string logPath = Path.Combine(
                    Application.StartupPath,
                    "certificate.log");

                File.AppendAllText(
                    logPath,
                    $"[{DateTime.Now}] {message}{Environment.NewLine}");
            }
            catch
            {
                // Ignore logging errors
            }
        }

        private void SetCertificate(X509Certificate2 certificate)
        {
            if (certificate == null)
            {
                MessageBox.Show(
                    this,
                    "Invalid certificate.",
                    "Certificate Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error);

                return;
            }

            _certificate = certificate;

            txtDetails.Text = _certificate.ToString(false);

            btnSave.Enabled = true;

            Log("Certificate loaded successfully.");
        }

        private void btnCreate_Click(object sender, EventArgs e)
        {
            try
            {
                Cursor = Cursors.WaitCursor;

                var cert = CertificateHelper.CreateCertificateAuthority(
                    "Quasar Server CA",
                    4096);

                SetCertificate(cert);

                MessageBox.Show(
                    this,
                    "Certificate created successfully.",
                    "Success",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information);

                Log("New certificate created.");
            }
            catch (Exception ex)
            {
                Log($"Certificate creation failed: {ex}");

                MessageBox.Show(
                    this,
                    $"Certificate creation failed:\n{ex.Message}",
                    "Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error);
            }
            finally
            {
                Cursor = Cursors.Default;
            }
        }

        private void btnImport_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog ofd = new OpenFileDialog())
            {
                ofd.Filter = "PKCS#12 Certificate (*.p12)|*.p12";
                ofd.CheckFileExists = true;
                ofd.Multiselect = false;
                ofd.InitialDirectory = Application.StartupPath;

                if (ofd.ShowDialog(this) == DialogResult.OK)
                {
                    try
                    {
                        Cursor = Cursors.WaitCursor;

                        var cert = new X509Certificate2(
                            ofd.FileName,
                            "",
                            X509KeyStorageFlags.Exportable |
                            X509KeyStorageFlags.PersistKeySet);

                        if (!cert.HasPrivateKey)
                        {
                            MessageBox.Show(
                                this,
                                "Certificate has no private key.",
                                "Import Error",
                                MessageBoxButtons.OK,
                                MessageBoxIcon.Warning);

                            return;
                        }

                        SetCertificate(cert);

                        MessageBox.Show(
                            this,
                            "Certificate imported successfully.",
                            "Success",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Information);

                        Log($"Certificate imported: {ofd.FileName}");
                    }
                    catch (CryptographicException ex)
                    {
                        Log($"Crypto error: {ex}");

                        MessageBox.Show(
                            this,
                            $"Invalid or corrupted certificate:\n{ex.Message}",
                            "Import Error",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error);
                    }
                    catch (Exception ex)
                    {
                        Log($"Import failed: {ex}");

                        MessageBox.Show(
                            this,
                            $"Error importing certificate:\n{ex.Message}",
                            "Import Error",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error);
                    }
                    finally
                    {
                        Cursor = Cursors.Default;
                    }
                }
            }
        }

        private void btnSave_Click(object sender, EventArgs e)
        {
            try
            {
                if (_certificate == null)
                {
                    MessageBox.Show(
                        this,
                        "Please create or import a certificate first.",
                        "Save Error",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Warning);

                    return;
                }

                if (!_certificate.HasPrivateKey)
                {
                    MessageBox.Show(
                        this,
                        "Certificate has no associated private key.",
                        "Save Error",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Warning);

                    return;
                }

                string directory = Path.GetDirectoryName(Settings.CertificatePath);

                if (!Directory.Exists(directory))
                    Directory.CreateDirectory(directory);

                byte[] certData = _certificate.Export(X509ContentType.Pkcs12);

                File.WriteAllBytes(Settings.CertificatePath, certData);

                Log($"Certificate saved: {Settings.CertificatePath}");

                MessageBox.Show(
                    this,
                    "Certificate saved successfully.\nPlease keep a secure backup.",
                    "Success",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information);

                Process.Start(new ProcessStartInfo
                {
                    FileName = "explorer.exe",
                    Arguments = $"/select,\"{Settings.CertificatePath}\"",
                    UseShellExecute = true
                });

                DialogResult = DialogResult.OK;

                Close();
            }
            catch (UnauthorizedAccessException ex)
            {
                Log($"Access denied: {ex}");

                MessageBox.Show(
                    this,
                    "Access denied.\nRun the application as administrator.",
                    "Permission Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error);
            }
            catch (Exception ex)
            {
                Log($"Save failed: {ex}");

                MessageBox.Show(
                    this,
                    $"Save failed:\n{ex.Message}",
                    "Save Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error);
            }
        }

        private void btnExit_Click(object sender, EventArgs e)
        {
            Close();
        }![1000168745](https://github.com/user-attachments/assets/4f8fb88e-c989-47c4-a1e5-1c1365d5cc17)

    }
}
