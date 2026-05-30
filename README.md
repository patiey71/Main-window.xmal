# Main-window.xmal
using System;
using System.Media;
using System.Windows;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;

namespace CybersecurityChatbot
{
    public partial class MainWindow : Window
    {
        // ── Engine ───────────────────────────────
        private readonly ChatbotEngine _engine = new ChatbotEngine();

        // ── Colours ──────────────────────────────
        private static readonly SolidColorBrush BotNameBrush   = new SolidColorBrush(Color.FromRgb(0,  212, 255));  // cyan
        private static readonly SolidColorBrush UserNameBrush  = new SolidColorBrush(Color.FromRgb(233, 69, 96));   // red
        private static readonly SolidColorBrush BotTextBrush   = new SolidColorBrush(Color.FromRgb(200, 220, 255));
        private static readonly SolidColorBrush UserTextBrush  = new SolidColorBrush(Colors.White);
        private static readonly SolidColorBrush TimestampBrush = new SolidColorBrush(Color.FromRgb(120, 130, 150));

        public MainWindow()
        {
            InitializeComponent();
            AsciiArtBlock.Text = ChatbotEngine.GetAsciiArt();
            PlayVoiceGreeting();
            AddBotMessage("Hello! 👋 Welcome to CyberGuard Assistant. I'm here to help you stay safe online.\n\nWhat's your name?");
        }

        // ════════════════════════════════════════
        //  Voice greeting
        // ════════════════════════════════════════
        private void PlayVoiceGreeting()
        {
            try
            {
                // Uses built-in Windows TTS via SpeechSynthesizer
                var synth = new System.Speech.Synthesis.SpeechSynthesizer();
                synth.Volume = 80;
                synth.Rate   = -1;
                synth.SpeakAsync("Welcome to CyberGuard Assistant. Your cybersecurity companion is ready.");
            }
            catch
            {
                // TTS not available on this machine — silently skip
            }
        }

        // ════════════════════════════════════════
        //  Event handlers
        // ════════════════════════════════════════
        private void SendButton_Click(object sender, RoutedEventArgs e) => ProcessInput();

        private void UserInputBox_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Key == Key.Enter) ProcessInput();
        }

        private void ClearButton_Click(object sender, RoutedEventArgs e)
        {
            ChatDisplay.Document.Blocks.Clear();
            AddBotMessage("Chat cleared. How can I help you with cybersecurity? 🛡️");
        }

        // ════════════════════════════════════════
        //  Core processing
        // ════════════════════════════════════════
        private void ProcessInput()
        {
            string input = UserInputBox.Text.Trim();
            if (string.IsNullOrEmpty(input)) return;

            AddUserMessage(input);
            UserInputBox.Clear();

            string response = _engine.GetResponse(input);
            AddBotMessage(response);

            // Auto-scroll to bottom
            ChatScrollViewer.ScrollToBottom();
        }

        // ════════════════════════════════════════
        //  Chat display helpers
        // ════════════════════════════════════════
        private void AddUserMessage(string text)
        {
            AppendMessage("You", text, UserNameBrush, UserTextBrush);
        }

        private void AddBotMessage(string text)
        {
            AppendMessage("CyberGuard 🛡️", text, BotNameBrush, BotTextBrush);
        }

        private void AppendMessage(string sender, string text, SolidColorBrush nameBrush, SolidColorBrush textBrush)
        {
            var doc = ChatDisplay.Document;

            // ── Sender + timestamp ───────────────
            var headerPara = new Paragraph { Margin = new Thickness(0, 8, 0, 0) };

            var senderRun = new Run(sender + "  ")
            {
                Foreground = nameBrush,
                FontWeight = FontWeights.Bold,
                FontSize   = 13
            };

            var timeRun = new Run(DateTime.Now.ToString("HH:mm"))
            {
                Foreground = TimestampBrush,
                FontSize   = 10
            };

            headerPara.Inlines.Add(senderRun);
            headerPara.Inlines.Add(timeRun);
            doc.Blocks.Add(headerPara);

            // ── Message text ─────────────────────
            var msgPara = new Paragraph { Margin = new Thickness(0, 2, 0, 0) };
            var msgRun = new Run(text)
            {
                Foreground = textBrush,
                FontSize   = 13
            };
            msgPara.Inlines.Add(msgRun);
            doc.Blocks.Add(msgPara);

            // ── Divider ──────────────────────────
            var divPara = new Paragraph { Margin = new Thickness(0) };
            var divRun  = new Run(new string('─', 60))
            {
                Foreground = new SolidColorBrush(Color.FromRgb(40, 50, 80)),
                FontSize   = 10
            };
            divPara.Inlines.Add(divRun);
            doc.Blocks.Add(divPara);
        }
    }
}
